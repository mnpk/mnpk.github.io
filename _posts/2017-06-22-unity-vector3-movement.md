---
layout: post
title: Unity3D Vector3 MoveTowards vs. Lerp vs. Slerp vs. SmoothDamp
---
Unity에서 한 점을 다른위치로 점차적으로 옮길 수 있는 방법은 여러가지가 있다. 그 중 `MoveTowards`, `Lerp`, `Slerp`, `SmoothDamp`에 대해 정리해보자.


# Vector3.MoveToward
https://docs.unity3d.com/ScriptReference/Vector3.MoveTowards.html

```cs
public static Vector3 MoveTowards(Vector3 current, Vector3 target, float maxDistanceDelta);
```

> Moves a point current in a straight line towards a target point.
The value returned by this function is a point maxDistanceDelta units closer to a target/ point along a line between current and target. If the target is closer than maxDistanceDelta/ then the returned value will be equal to target (ie, the movement will not overshoot the target). Negative values of maxDistanceDelta can be used to push the point away from the target.


current 점에서 target 점으로 직선으로 maxDistanceDelta 만큼 간 값을 리턴한다.
current에서 target 거리가 maxDistanceDelta보다 작으면 target이 리턴된다.
maxDistanceDelta에 deltaTime에 비례하는 값을 넣으면 시간에 따라 동일한 거리만큼 이동되므로 이동 속도는 일정하다.
가장 정직하게 일정한 거리만큼 이동할 수 있는 방법이다.



# Vector3.Lerp
https://docs.unity3d.com/ScriptReference/Vector3.Lerp.html
```cs
public static Vector3 Lerp(Vector3 a, Vector3 b, float t);
```

> Linearly interpolates between two vectors.
Interpolates between the vectors a and b by the interpolant t. The parameter t is clamped to the range [0, 1]. This is most commonly used to find a point some fraction of the way along a line between two endpoints (e.g. to move an object gradually between those points).

> When t = 0 returns a. When t = 1 returns b. When t = 0.5 returns the point midway between a and b.


두 벡터 a와 b 사이를 t비율만큼 선형으로 interpolate 한다. 파라미터 t는 0~1 사이 값이며 a와 b 사이의 위치 비율이다. 0~1을 넘으면 0~1사이로 잘린다. 0을 넣으면 a가 리턴되고 1을 넣으면 b가 리턴된다. 0.5를 넣으면 a와 b의 한 가운데가 리턴된다.
a에 현재위치, b에 목표위치, t를 deltaTime에 비례하는 값을 넣게 되면 현재 위치부터 목표지점까지 일정 비율 만큼 이동하게 되는데 목표 위치까지의 거리는 이동한 만큼 줄어들게 되므로 결과적으로 이동 속도는 처음이 가장 빠르고 점점 느려지게 된다.
예를 들어서(0,0,0)에서 출발하여 a=현재위치, b=(10,0,0), t=0.1이고 1초마다 Lerp를 계산하게 되면
처음에는 0→10의 10%인 1만큼 이동하여 (1,0,0)이 되고
두번째는 1→10의 10%인 0.9만큼 이동
세번째는 1.9→10의 10%인 0.81만큼 이동
...
이렇게 남은 거리의 동일한 비율만큼 이동하므로 실제 한번에 이동하는 거리, 즉 속도는 처음이 가장 빠르고 목표지점에 가까워 질수록 느리다.


# Vector3.Slerp

https://docs.unity3d.com/ScriptReference/Vector3.Slerp.html
```cs
public static Vector3 Slerp(Vector3 a, Vector3 b, float t);
```

> Spherically interpolates between two vectors.
Interpolates between a and b by amount t. The difference between this and linear interpolation (aka, "lerp") is that the vectors are treated as directions rather than points in space. The direction of the returned vector is interpolated by the angle and its magnitude is interpolated between the magnitudes of from and to.

> The parameter t is clamped to the range [0, 1].


두 점 사이를 구 형태로 interpolate한다. 파라미터 t는 Lerp와 동일하게 0~1 사이의 값이며 a와 b사이에 interpolate할 비율이다. Lerp가 a, b를 공간상의 점으로 보고 둘 사이를 직선으로 이어서 t비율 만큼을 이동한 점을 찾는 것이라고 보면, Slerp는 두 벡터 a, b를 Direction으로 보고 두 Direction 사이의 중간 Direction을 찾는 것이라고 볼 수 있다. 

빨간점 = a
초록점 = b
파란점 = Lerp(a, b, 0.5)
하얀점 = Slerp(a, b, 0.5)

![](http://upload-images.jianshu.io/upload_images/289095-de2dda14c1f63776.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Slerp(현재 위치, 목표위치, deltaTime * 비율) 형태로 사용하게 되면 기본적으로 Lerp와 비슷하나 3D인 경우 직선이 아닌 구형으로 이동하게 되고, 따라서 직선 방향으로의 이동속도는 Lerp에 삼각함수 곡선이 곱해진 형태로  Lerp보다 초반 후반이 더 느리고 중간이 더 빠른 형태가 된다.

![](http://www.faustofonseca.com/wp-content/uploads/chart.png)


# Vector3.SmoothDamp
https://docs.unity3d.com/ScriptReference/Vector3.SmoothDamp.html
```cs
public static Vector3 SmoothDamp(Vector3 current, Vector3 target, ref Vector3 currentVelocity, float smoothTime, float maxSpeed = Mathf.Infinity, float deltaTime = Time.deltaTime);
```

> Gradually changes a vector towards a desired goal over time.
The vector is smoothed by some spring-damper like function, which will never overshoot. The most common use is for smoothing a follow camera.


벡터를 목표 지점까지시간의 흐름에 비례해 점차적으로 부드럽게 이동시킨다. 카메라를 부드럽게 따라가게 하는데 가장 많이 사용된다. 현재 속도를 ref로 넘긴다. 초반 속도가 0이었을 때 SmoothDamp를 이용해 이동하면 속도는 다음 그래프 파란 선과 같은 모습이 된다.

![](https://i1.wp.com/devblog.aliasinggames.com/wp-content/uploads/2016/03/smoothDamp.png)




# Comparison

위 4가지를 이용해 오브젝트를 이동시키는 모습을 한 번에 비교해보자.
아래 움짤에서 각 구는 위쪽부터 SmoothDamp, Lerp, Slerp, MoveTowrad를 사용해 움직인다.
각 구의 목표점은 x축만 -5에서 5로 변경했다.

![](http://i.imgur.com/FeKRE1c.gif)

