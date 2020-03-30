# Full GC(Ergonomics)产生的原因

发生Full GC的原因有很多，不只Allocation Failure。

还有以下很多种原因：

```c++
#include "precompiled.hpp"
#include "gc/shared/gcCause.hpp"

const char* GCCause::to_string(GCCause::Cause cause) {
  switch (cause) {
    case _java_lang_system_gc:
      return "System.gc()";

    case _full_gc_alot:
      return "FullGCAlot";

    case _scavenge_alot:
      return "ScavengeAlot";

    case _allocation_profiler:
      return "Allocation Profiler";

    case _jvmti_force_gc:
      return "JvmtiEnv ForceGarbageCollection";

    case _gc_locker:
      return "GCLocker Initiated GC";

    case _heap_inspection:
      return "Heap Inspection Initiated GC";

    case _heap_dump:
      return "Heap Dump Initiated GC";

    case _wb_young_gc:
      return "WhiteBox Initiated Young GC";

    case _wb_conc_mark:
      return "WhiteBox Initiated Concurrent Mark";

    case _wb_full_gc:
      return "WhiteBox Initiated Full GC";

    case _update_allocation_context_stats_inc:
    case _update_allocation_context_stats_full:
      return "Update Allocation Context Stats";

    case _no_gc:
      return "No GC";

    case _allocation_failure:
      return "Allocation Failure";

    case _tenured_generation_full:
      return "Tenured Generation Full";

    case _metadata_GC_threshold:
      return "Metadata GC Threshold";

    case _metadata_GC_clear_soft_refs:
      return "Metadata GC Clear Soft References";

    case _cms_generation_full:
      return "CMS Generation Full";

    case _cms_initial_mark:
      return "CMS Initial Mark";

    case _cms_final_remark:
      return "CMS Final Remark";

    case _cms_concurrent_mark:
      return "CMS Concurrent Mark";

    case _old_generation_expanded_on_last_scavenge:
      return "Old Generation Expanded On Last Scavenge";

    case _old_generation_too_full_to_scavenge:
      return "Old Generation Too Full To Scavenge";

    case _adaptive_size_policy:
      return "Ergonomics";

    case _g1_inc_collection_pause:
      return "G1 Evacuation Pause";

    case _g1_humongous_allocation:
      return "G1 Humongous Allocation";

    case _dcmd_gc_run:
      return "Diagnostic Command";

    case _last_gc_cause:
      return "ILLEGAL VALUE - last gc cause - ILLEGAL VALUE";

    default:
      return "unknown GCCause";
  }
  ShouldNotReachHere();
}
```

由上篇担保机制可以看到，在server模式（即Parallel Scavenge+Serial Old）组合下会发生Full GC(Ergonomics)，而client模式（即Serial GC收集器组合）不会发生full GC。

Parallel Scavenge回收策略的源码片段：

```c++
// This method contains all heap specific policy for invoking scavenge.
// PSScavenge::invoke_no_policy() will do nothing but attempt to
// scavenge. It will not clean up after failed promotions, bail out if
// we've exceeded policy time limits, or any other special behavior.
// All such policy should be placed here.
//
// Note that this method should only be called from the vm_thread while
// at a safepoint!
bool PSScavenge::invoke() {
  assert(SafepointSynchronize::is_at_safepoint(), "should be at safepoint");
  assert(Thread::current() == (Thread*)VMThread::vm_thread(), "should be in vm thread");
  assert(!ParallelScavengeHeap::heap()->is_gc_active(), "not reentrant");

  ParallelScavengeHeap* const heap = ParallelScavengeHeap::heap();
  PSAdaptiveSizePolicy* policy = heap->size_policy();
  IsGCActiveMark mark;

  const bool scavenge_done = PSScavenge::invoke_no_policy();
  const bool need_full_gc = !scavenge_done ||
    policy->should_full_GC(heap->old_gen()->free_in_bytes());
  bool full_gc_done = false;

  if (UsePerfData) {
    PSGCAdaptivePolicyCounters* const counters = heap->gc_policy_counters();
    const int ffs_val = need_full_gc ? full_follows_scavenge : not_skipped;
    counters->update_full_follows_scavenge(ffs_val);
  }

  if (need_full_gc) {
    GCCauseSetter gccs(heap, GCCause::_adaptive_size_policy);
    CollectorPolicy* cp = heap->collector_policy();
    const bool clear_all_softrefs = cp->should_clear_all_soft_refs();

    if (UseParallelOldGC) {
      full_gc_done = PSParallelCompact::invoke_no_policy(clear_all_softrefs);
    } else {
      full_gc_done = PSMarkSweep::invoke_no_policy(clear_all_softrefs);
    }
  }

  return full_gc_done;
}
```

核心代码：

```c++
if (need_full_gc) {
    GCCauseSetter gccs(heap, GCCause::_adaptive_size_policy);
    CollectorPolicy* cp = heap->collector_policy();
    const bool clear_all_softrefs = cp->should_clear_all_soft_refs();

    if (UseParallelOldGC) {
        full_gc_done = PSParallelCompact::invoke_no_policy(clear_all_softrefs);
    } else {
        full_gc_done = PSMarkSweep::invoke_no_policy(clear_all_softrefs);
    }
}  
```

注：基本内容是如果需要full gc那么就进入if块，然后执行full gc逻辑。另外这里的_adaptive_size_policy 常量就是对应的Ergonomics：

```c++
case _adaptive_size_policy:
	return "Ergonomics";
```

**那么full gc的条件是什么呢？也就是什么情况导致发生了本次full gc呢？**

need_full_gc：

full gc条件：

```c++
const bool need_full_gc = !scavenge_done ||
  policy->should_full_GC(heap->old_gen()->free_in_bytes());
```

should_ful_GC方法：

```c++
// If the remaining free space in the old generation is less that
// that expected to be needed by the next collection, do a full
// collection now.
bool PSAdaptiveSizePolicy::should_full_GC(size_t old_free_in_bytes) {

    // A similar test is done in the scavenge's should_attempt_scavenge().  If
    // this is changed, decide if that test should also be changed.
    bool result = padded_average_promoted_in_bytes() > (float) old_free_in_bytes;
    //如果晋升到老年代的平均大小大于老年代的剩余大小，则认为要进行一次full gc

    log_trace(gc, ergo)(
        "%s after scavenge average_promoted " 
        SIZE_FORMAT " padded_average_promoted " 
        SIZE_FORMAT " free in old gen " SIZE_FORMAT,
        result ? "Full" : "No full",
        (size_t) average_promoted_in_bytes(),
        (size_t) padded_average_promoted_in_bytes(),
        old_free_in_bytes);
    return result;
}
```

通过查看should_full_GC方法，我们发现了这行代码：

```
bool result = padded_average_promoted_in_bytes() > (float) old_free_in_bytes;
```

**通过该行代码，我们知道，如果晋升到老生代的平均大小大于老生代的剩余大小，则会返回true，认为需要一次full gc。**

通过注释也可以知道：

```
If the remaining free space in the old generation is less than
that expected to be needed by the next collection, do a full
collection now.

如果老生代的剩余空间少于下一次收集所需的剩余空间，那么现在就做一个完整的收集。
```

如果 padded_average_promoted_in_bytes()大于老生代剩余空间，那么就返回true，表示要触发一次fullgc。

那么padded_average_promoted_in_bytes()这个平均大小是怎么算出来的呢？我们去看看：

```c++
// Padded average in bytes
size_t padded_average_promoted_in_bytes() const {
    return (size_t)_avg_promoted->padded_average();
}

float padded_average() const         { return _padded_avg; }
// A weighted average that includes a deviation from the average,
// some multiple of which is added to the average.
//
// This serves as our best estimate of an upper bound on a future
// unknown.
class AdaptivePaddedAverage : public AdaptiveWeightedAverage {
    private:
    float          _padded_avg;     // The last computed padded average
    float          _deviation;      // Running deviation from the average
    unsigned       _padding;        // A multiple which, added to the average,
    // gives us an upper bound guess.

    protected:
    void set_padded_average(float avg)  { _padded_avg = avg;  }
    void set_deviation(float dev)       { _deviation  = dev;  }

    public:
    AdaptivePaddedAverage() :
    AdaptiveWeightedAverage(0),
    _padded_avg(0.0), _deviation(0.0), _padding(0) {}

    AdaptivePaddedAverage(unsigned weight, unsigned padding) :
    AdaptiveWeightedAverage(weight),
    _padded_avg(0.0), _deviation(0.0), _padding(padding) {}

    // Placement support
    void* operator new(size_t ignored, void* p) throw() { return p; }
    // Allocator
    void* operator new(size_t size) throw() { return CHeapObj<mtGC>::operator new(size); }

    // Accessor
    float padded_average() const         { return _padded_avg; }
    float deviation()      const         { return _deviation;  }
    unsigned padding()     const         { return _padding;    }

    void clear() {
        AdaptiveWeightedAverage::clear();
        _padded_avg = 0;
        _deviation = 0;
    }

    // Override
    void  sample(float new_sample);

    // Printing
    void print_on(outputStream* st) const;
    void print() const;
};
```

可以从代码和注释中我们发现：

**加权平均值包括与平均值的偏差，其平均值加上其中的一些倍数。 这是对未来未知数的上限的最佳估计。**

也就是通过这样的算法，虚拟机估算出下次分配可能会发生无法分配的问题，于是**提前预测到可能的问题，提前发生一次full gc**。

于是这次full gc就发生了！

那么你也许有疑问说[Full GC (Ergonomics) 的Ergonomics究竟是个什么东东？

**Ergonomics翻译成中文，一般都是“人体工程学”。在JVM中的垃圾收集器中的Ergonomics就是负责自动的调解gc暂停时间和吞吐量之间的平衡，然后你的虚拟机性能更好的一种做法。**

对于注重吞吐量的收集器来说，在某个generation被过渡使用之前，GC ergonomics就会启动一次GC。

正如我们前面提到的，发生本次full gc正是在使用Parallel Scavenge收集器的情况下发生的。

而Parallel Scavenge正是一款注重吞吐量的收集器：

> Parallel  Scavenge的目标是达到一个可控的吞吐量，吞吐量=程序运行时间/（程序运行时间+GC时间），如程序运行了99s，GC耗时1s，吞吐量=99/（99+1）=99%。Parallel  Scavenge提供了两个参数用以精确控制吞吐量，分别是用以控制最大GC停顿时间的-XX:MaxGCPauseMillis及直接控制吞吐量的参数-XX:GCTimeRatio。