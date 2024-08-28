优先级实现



Lane 机制，每种任务优先级对应一个独特的二进制数，可以通过与运算的结果来匹配优先级

```js
export const NoLanes: Lanes = /*                        */ 0b0000000000000000000000000000000;
export const NoLane: Lane = /*                          */ 0b0000000000000000000000000000000;

export const SyncHydrationLane: Lane = /*               */ 0b0000000000000000000000000000001;
export const SyncLane: Lane = /*                        */ 0b0000000000000000000000000000010;
export const SyncLaneIndex: number = 1;

export const InputContinuousHydrationLane: Lane = /*    */ 0b0000000000000000000000000000100;
export const InputContinuousLane: Lane = /*             */ 0b0000000000000000000000000001000;

export const DefaultHydrationLane: Lane = /*            */ 0b0000000000000000000000000010000;
export const DefaultLane: Lane = /*                     */ 0b0000000000000000000000000100000;
......

export function getLabelForLane(lane: Lane): string | void {
    if (enableSchedulingProfiler) {
        if (lane & SyncHydrationLane) {
          return 'SyncHydrationLane';
        }
        if (lane & SyncLane) {
          return 'Sync';
        }
      ...
    }
  }
```



对于不同优先级的任务，分配不同的过期时间，优先级越高，过期时间越短，也就意味着需要先执行

```js
export const retryLaneExpirationMs = 5000;
export const syncLaneExpirationMs = 250;
export const transitionLaneExpirationMs = 5000;

function computeExpirationTime(lane: Lane, currentTime: number) {
    switch (lane) {
        case SyncHydrationLane:
        case SyncLane:
        case InputContinuousHydrationLane:
        case InputContinuousLane:
        	return currentTime + syncLaneExpirationMs;
        case DefaultHydrationLane:
        case DefaultLane:
        case TransitionHydrationLane:
        case TransitionLane1:
        ......
        case TransitionLane15:
        	return currentTime + transitionLaneExpirationMs;
    }
  }
```

