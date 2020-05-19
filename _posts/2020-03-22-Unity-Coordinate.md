---
layout: post
title:  "Unity学习—坐标系与空间变换"
description: Unity 的坐标系转换和物体的位移旋转
date:   2020-03-22 23:00:07 +0800
categories: [Unity]
tag: [学习笔记,坐标系,空间变换]

---

讲解的 Unity 中几种不同的坐标系与其之间的转换，以及汇总物体的移动和旋转方法

本文其他地址：[简书](https://www.jianshu.com/p/18e6163f43e4)	[知乎](https://zhuanlan.zhihu.com/p/115353437) 	[掘金](https://juejin.im/post/5e7776d76fb9a07ccb7ebb2d)  

## 坐标系  

### 坐标系种类  

Unity 中使用到的坐标系分为以下四种  

1. 世界坐标系  Word Space   

   -  即世界空间使用的坐标系，基本单位 unit，x 正方向：左向右， y 正方向：下向上，z 正方向：屏外向屏内，
   -  任何物体使用 Transform.position 即可获得世界坐标值  

   -  场景中根物体使用的就是世界坐标，可在 Inspector 查看世界坐标值

   -  对于非根物体则以父物体位置为原点位置使用**本地坐标系  Local Space**，即相对父物体位置，该物体 Inspector 数值为本地坐标值，可使用 Transform.localposition 获取本地坐标值

2. 屏幕坐标系 Screen Space  

   - 基本单位像素，屏幕左下角为（0，0），右上角为（Screen.width，Screen.height），即实际运行屏幕下的游戏窗口像素值，z 为相机世界坐标单位值
   - Input.mousePosition 获取的鼠标坐标，Input.GetTouch(0).position 获取触摸坐标  

3. 视口坐标系  Viewport Space  

   - 左下角为（0，0），右上角为（1，1），z 为相机世界坐标单位值
   - 适合用于坐标系转换  

4. UGUI 坐标系  UGUI Space 

   - 基本单位像素，屏幕左上角为（0，0），右下角为（Screen.width，Screen.height）

### 坐标系转换  

```c#  
// 本地→世界    
transform.TransformPoint(position);
// 世界→本地  
transform.InverseTransformPoint(position);
// 世界→屏幕  
camera.WorldToScreenPoint(position);  
// 世界→视口  
Camera.main.WorldToViewportPoint(position)
// 屏幕→视口  
camera.ScreenToViewportPoint(position);
// 视口→屏幕  
camera.ViewportToScreenPoint(position);
// 视口→世界  
camera.ViewportToWorldPoint(position);
```

## 空间变换  

### 坐标系选择  

世界坐标系 Space.World 与自身坐标系 Space.Self  

Vector3.forward、Vector3.back、Vector3.left、Vector3.right、Vector3.up、Vector3.down 数值固定

transform.forward、 transform.right、transform.up 数值不定，依据物体自身旋转变化，如 transform.forward 为物体 z 轴在世界坐标系中所指方向   

### 非刚体变换   

非刚体物体在移动或旋转时不检测碰撞   

#### 非刚体移动   

1. Transform.positon  世界位置坐标  

2. Transform.localPostion  本地位置坐标

3. Transform.Translate(Vector3 translation, Space relativeTo = Space.Self)  

4. Transform.Translate(float x, float y, float z, Space relativeTo = Space.Self)  

5. Transform.Translate(Vector3 translation, Transform relativeTo)  

6. Transform.Translate(float x, float y, float z, Transform relativeTo)

   ```c#
   void Update()
   {
        // 以每秒1个单位速度延世界坐标系y轴正方向移动
        transform.Translate(Vector3.up * Time.deltaTime, Space.World);
        // 以每秒1个单位速度延主相机本地坐标系x轴正方向移动
        transform.Translate(Vector3.right * Time.deltaTime, Camera.main.transform);
   }
   ```

7. Vector3.MoveTowards(Vector3 current, Vector3 target, float maxDistanceDelta)  

   以一定速度向目标移动直至到达目标位置，对比插值法可限制最大速度

   ```c#
   // maxDistanceDelta 最大移动距离
   void Update()
   {
        // 当前帧移动距离
        float step =  speed * Time.deltaTime; 
        // 由当前位置向目标位置移动距离 step 得出新位置，超过终点则返回终点值  
        transform.position = Vector3.MoveTowards(transform.position, target.position, step);
   }
   ```

8. Vector3.SmoothDamp(Vector3 current, Vector3 target, ref Vector3 currentVelocity, float smoothTime, float maxSpeed = Mathf.Infinity, float deltaTime = Time.deltaTime)   

   实现平滑移动，可控制速度，一般用于摄像机跟随  

   ```c#
   // 通过使用自身修改的速度指针计算当前位置
   private Vector3 velocity = Vector3.zero;
   void Update()
   {
        // 定义目标物体的后上方为目标位置
        Vector3 targetPosition = target.TransformPoint(new Vector3(0, 5, -10));
        // 以每帧变化的速度 velocity 由当前位置向目标位置移动
        transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref velocity, 0.3F);
   }
   ```

9. Vector3.Lerp(Vector3 a, Vector3 b, float t)   

10. Vector3.LerpUnclamped(Vector3 a, Vector3 b, float t)  

11. Vector3.Slerp(Vector3 a, Vector3 b, float t)  

12. Vector3.SlerpUnclamped(Vector3 a, Vector3 b, float t)  

    插值法，适用于已知起点与终点的情况   

    ```c#
    void Update()
    {
        // 当前时间已移动距离
        float distCovered = (Time.time - startTime) * speed;
        // 已移动距离与总距离比例
        float fractionOfJourney = distCovered / journeyLength;  
        // 对起点终点之间插值取相应比例值，范围 0.0-1.0 即 startMarker.position - endMarker.position
        transform.position = Vector3.Lerp(startMarker.position, endMarker.position, fractionOfJourney);
    }  
    
    void Update()
    {
        // 弧心
        Vector3 center = (sunrise.position + sunset.position) * 0.5F;
        // 下移弧心使弧垂直
        center -= new Vector3(0, 1, 0);
        // 对弧插值
        Vector3 riseRelCenter = sunrise.position - center;
        Vector3 setRelCenter = sunset.position - center;
        // 已过时长占总时长比例
        float fracComplete = (Time.time - startTime) / journeyTime;
        transform.position = Vector3.Slerp(riseRelCenter, setRelCenter, fracComplete);
        transform.position += center;
    }
    
    
    ```
    
    ```c#
    // 线性插值法计算两点确定直线任意位置 无范围限制
    Vector3.LerpUnclamped(startPostion, targetPosition, 3.2f);  
    // 球形插值，插值结果为由两向量间角度和长度插值得到的向量值，常用于计算太阳轨迹
    Vector3.Slerp(startPostion, targetPosition, 0.3f);
    ```
    
    ```c#
    // 数学插值
    gameObject.transform.localPosition = new Vector3(
    Mathf.Lerp(startPostion.x, targetPosition.x, MoveSpeed * Time.deltaTime),
    Mathf.Lerp(startPostion.y, targetPosition.y, MoveSpeed * Time.deltaTime),
    Mathf.Lerp(startPostion.z, targetPosition.z, MoveSpeed * Time.deltaTime));
    ```

#### 非刚体旋转  

旋转需使用旋转方法，不要直接修改属性值，当超过360会发生错误  

1. Transform.eulerAngles   欧拉角  

2. Transform.localEulerAngles 本地欧拉角

3. Transform.rotation  四元数

4. Transform.Rotate(Vector3 eulers, Space relativeTo = Space.Self)  

5. Transform.Rotate(float xAngle, float yAngle, float zAngle, Space relativeTo = Space.Self)  

6. Transform.Rotate(Vector3 axis, float angle, Space relativeTo = Space.Self) 

   ```c#
   void Update()
   {
        // 以物体 y 轴正方向以30°每秒的速度旋转
        transform.Rotate(Vector3.up * 30 * Time.deltaTime, Space.Self);
   }
   ```

7. Transform.RotateAround(Vector3 point, Vector3 axis, float angle)  

   ```c#
   void Update()
   {
        // 绕世界坐标系中目标位置的 y 轴正方向以30°每秒旋转和移动（即绕点画圆）
        transform.RotateAround(target, Vector3.up, 30 * Time.deltaTime);
   }
   ```

8. Transform.LookAt(Vector3 worldPosition, Vector3 worldUp = Vector3.up)

9. Transform.LookAt(Transform target, Vector3 worldUp = Vector3.up)  

   以y为轴旋转，使z轴指向目标  

   ```c#
   // 旋转使物体 z 轴指向目标位置，且 x 轴同目标方向与 upwards 的叉积方向一致，y 轴同 z 和 x 轴的叉积方向一致
   transform.LookAt(targetPosition, Vector3.up);
   transform.LookAt(target.transform, Vector3.up);
   ```

10. Vector3.RotateTowards(Vector3 current, Vector3 target, float maxRadiansDelta, float maxMagnitudeDelta)   

    ```c#
    //maxRadiansDelta 旋转最大弧度差
    //maxMagnitudeDelta 旋转最大长度差
    void Update()
    {
        // 确定旋转方向
        Vector3 targetDirection = target.position - transform.position;
        // 一帧旋转角度
        float singleStep = speed * Time.deltaTime;  
      	// 将物体 z 轴正方向向目标方向旋转一帧角度，得到当前帧方向
        Vector3 newDirection = Vector3.RotateTowards(transform.forward, targetDirection, singleStep, 0.0f);
        // 计算由 z 轴正方向旋转到当前帧旋转方向的四元数角度
        transform.rotation = Quaternion.LookRotation(newDirection);
    }
    ```

11. Quaternion.RotateTowards(Quaternion from, Quaternion to, float maxDegreesDelta)     

    ```c#
    void Update()
    {
        // 当前一帧旋转角度
        var step = speed * Time.deltaTime;
        // 由当前旋转角度向目标旋转角度旋转一帧角度
        transform.rotation = Quaternion.RotateTowards(transform.rotation, target.rotation, step);
    }
    ```

12. Quaternion.FromToRotation(Vector3 fromDirection, Vector3 toDirection)  

    ```c#  
    void Start()
    {
        // 将物体 y 轴旋转至当前 z 轴正方向
        transform.rotation = Quaternion.FromToRotation(Vector3.up, transform.forward);
    }
    ```

13. Quaternion.LookRotation(Vector3 forward, Vector3 upwards = Vector3.up)   

    若 forward 或 upwards 大小为0则返回0  

    若 forward 与 upwards 共线则返回0

    ```c#
    void Update()
    {
        // 目标方向
        Vector3 relativePos = target.position - transform.position;
        // 计算由 z 轴正方向旋转到目标方向的角度，且 x 轴同目标方向与 upwards 的叉积方向一致，y 轴同 z 和 x 轴的叉积方向一致
        Quaternion rotation = Quaternion.LookRotation(relativePos, Vector3.up);
        transform.rotation = rotation;
    }
    ```

14. Quaternion.AngleAxis(float angle, Vector3 axis)    

    ```c#
    void Start()
    {
        // 绕世界坐标系 y 轴正方向旋转30°
        transform.rotation = Quaternion.AngleAxis(30, Vector3.up);
    }
    ```

15. Quaternion.Lerp(Quaternion a, Quaternion b, float t)

16. Quaternion.LerpUnclamped(Quaternion a, Quaternion b, float t)

17. Quaternion.Slerp(Quaternion a, Quaternion b, float t)  

18. Quaternion.SlerpUnclamped(Quaternion a, Quaternion b, float t)  

    ```c#
    // 基本同上述 Vector3.Lerp
    void Update()
    {
        transform.rotation = Quaternion.Lerp(from.rotation, to.rotation, Time.time * speed);  
        transform.rotation = Quaternion.Slerp(from.rotation, to.rotation, timeCount);
        timeCount = timeCount + Time.deltaTime;
    }  
    
    void Update()
    {
        transform.rotation = Quaternion.Slerp(from.rotation, to.rotation, timeCount);
        timeCount = timeCount + Time.deltaTime;
    }
    ```

### 刚体变换  

刚体运动会计算碰撞，不会发生碰撞体嵌入问题，开启 Kinematic 则不受力、碰撞等物理作用，对 Rigidbody 的操作都应在 `FixUpdate` 中进行

#### 刚体移动  

1. Rigidbody.position   刚体世界位置坐标

2. Rigidbody.velocity    刚体速度

3. Rigidbody.MovePosition(Vector3 position)    

   与 Rigidbody.interpolation 设置共同作用，开启 `interpolation` 则刚体插值平滑过渡到目标位置，移动时检测碰撞， 

   ```c#
   void FixedUpdate()
   {
        Rigidbody rb = GetComponent<Rigidbody>();
        // 刚体向右移动
        rb.MovePosition(transform.position + transform.right * Time.fixedDeltaTime);
   }
   ```

4. AddForce(Vector3 force, ForceMode mode = ForceMode.Force)    

   作用力参考世界坐标系

5. AddForce(float x, float y, float z, ForceMode mode = ForceMode.Force)    

6. AddRelativeForce(Vector3 force, ForceMode mode = ForceMode.Force)    

   作用力参考本地坐标系

7. AddRelativeForce(float x, float y, float z, ForceMode mode = ForceMode.Force)   

8. AddForceAtPosition(Vector3 force, Vector3 position, ForceMode mode = ForceMode.Force)    

   在点上施加力，会给物体加上扭力

9. AddExplosionForce(float explosionForce, Vector3 explosionPosition, float explosionRadius, float upwardsModifier  = 0.0f, ForceMode mode = ForceMode.Force))    

   ```c#
   void FixedUpdate()
   {
        Rigidbody rb = GetComponent<Rigidbody>();
        // 物体 z 轴正方向施加1个单位的力
        rb.AddForce(transform.forward * 1.0f);
        // 物体 z 轴正方向施加1个单位的冲力
        rb.AddForce(0, 0, 1.0f, ForceMode.Impulse);  
        // 物体 z 轴正方向施加1个单位的力
        rb.AddRelativeForce(Vector3.forward * 1.0f);
     
        // 模拟爆炸力，由爆炸中心向物体中心施加1个单位力
        Vector3 direction = from.transform.position - transform.position;
        body.AddForceAtPosition(direction.normalized, transform.position);
   }
   ```

#### 刚体旋转  

1. Rigidbody.rotation  刚体旋转

2. Rigidbody.angularVelocity  刚体角速度

3. Rigidbody.constraints   刚体约束   

   约束刚体在某一或多轴的移动或旋转

   ```c#
   // 限制刚体在 z 轴的移动和旋转
   m_Rigidbody.constraints = RigidbodyConstraints.FreezePositionZ | RigidbodyConstraints.FreezeRotationZ;
   ```

4. Rigidbody.MoveRotation(Quaternion rot)   

   与 Rigidbody.interpolation 设置共同作用，则刚体插值平滑旋转到目标角度  

   ```c#
   Quaternion deltaRotation = Quaternion.Euler(m_EulerAngleVelocity * Time.deltaTime);
   m_Rigidbody.MoveRotation(m_Rigidbody.rotation * deltaRotation);
   ```

5. Rigidbody.AddTorque(Vector3 torque, ForceMode mode = ForceMode.Force)  

6. Rigidbody.AddTorque(float x, float y, float z, ForceMode mode = ForceMode.Force)  

7. Rigidbody.AddRelativeTorque(Vector3 torque, ForceMode mode = ForceMode.Force)  

8. Rigidbody.AddRelativeTorque(float x, float y, float z, ForceMode mode = ForceMode.Force)  

   ```c#
   void FixedUpdate()
   {
        Rigidbody rb = GetComponent<Rigidbody>();
        // 根据输入水平数据绕物体 y 轴正方向添加扭力
        float turn = Input.GetAxis("Horizontal");
        rb.AddTorque(transform.up * torque * turn);
        // 物体 y 轴正方向添加扭力
        rb.AddRelativeTorque(Vector3.up * torque * turn);
   }
   ```

