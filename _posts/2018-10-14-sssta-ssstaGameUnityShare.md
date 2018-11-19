---
layout: post
title:  "SSSTA-Game组Unity分享大纲"
date:   2018-10-14
excerpt: "SSSTAGame组18级Unity分享的大纲（长期更新）"
tag:
- sssta
comments: true
---

## 第一次分享

#### Unity的窗口

顶部文件、编辑、资源、GO、组件、窗口、帮助

操作工具、三个按钮、右侧选项

Hierarchy,Scene,Game,Inspector,Project,Console,Services

#### 建立一个立方体

建立它、移动它、更换它的颜色、给它附加一个摄像机、给它附加一个脚本

#### 通过脚本使立方体移动

使用public封装，在Inspector中改变它的值

真厉害！

示例代码：
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class move : MonoBehaviour {

	public float speed;
	public GameObject capsule;
	// Use this for initialization
	void Start () {
		
	}
	
	// Update is called once per frame
	void Update () {
		
		if(Input.GetKey(KeyCode.A))
			capsule.transform.Translate(new Vector3(-0.1f*speed,0,0));
		if(Input.GetKey(KeyCode.D))
			capsule.transform.Translate(new Vector3(0.1f*speed,0,0));
		if(Input.GetKey(KeyCode.W))
			capsule.transform.Translate(new Vector3(0,0.1f*speed,0));
		if(Input.GetKey(KeyCode.S))
			capsule.transform.Translate(new Vector3(0,-0.1f*speed,0));
	}
}
```


## 第二次分享

### 一个更优雅的移动方式

上次分享提到了一种最简单的移动，但是这样写移动有一个问题。即同时按住两个方向上移动的键时，物体会以原先速度的根号二倍速移动。这个问题之所以发生是因为同时满足了两个if的条件，故而速度相加的斜向速度为原来的根号二倍速。此时，我们就需要一个优雅的解决方案了。

显而易见，一个可以解决问题的简单方法是if else结构。即把前后左右斜向移动使用一个if以及七个if else语句实现。

还可以在rigidbody组件上给物体添加力来实现移动。
方法为rigidbody.Addforce。

```
（1）ForceMode.Force：默认方式，使用刚体的质量计算，以每帧间隔时间为单位计算动量。设FixedUpdate()的执行频率采用系统默认值（即0.02s），
则由动量定理f•t=m•v
可得：10*0.02=2*v1，从而可得v1=0.1，即每帧刚体在X轴上值增加0.1米，从而可计算得刚体的每秒移动速度为v2=(1/0.02)*v1=5m/s。
（2）ForceMode.Acceleration：在此种作用方式下会忽略刚体的实际质量而采用默认值1.0f，时间间隔以系统帧频间隔计算（默认值为0.02s），
即f•t=1.0•v
即可得v1= f•t=10*0.02=0.2，即刚体每帧增加0.2米，从而可得刚体的每秒移动速度为v2=(1/0.02)*v1=10m/s。
（3）ForceMode.Impulse：此种方式采用瞬间力作用方式，即把t的值默认为1，不再采用系统的帧频间隔，
即f•1.0=m•v
即可得v1=f/m=10.0/2.0=5.0，
即刚体每帧增加5.0米，从而可得刚体每秒的速度为v2=(1/0.02)*5.0=250m/s。
（4）ForceMode.VelocityChange：此种作用方式下将忽略刚体的实际质量，采用默认质量1.0，同时也忽略系统的实际帧频间隔，采用默认间隔1.0，
即f•1.0=1.0•v
即可得v1=f=10.0，
即刚体每帧沿X轴移动距离为10米，从而可得刚体每秒的速度为v2=(1/0.02)*v1=500m/s。
```
也可以使用CharacterController组件的Move以及SimpleMove。参考unity示例代码:

[传送门](https://docs.unity3d.com/ScriptReference/CharacterController.html)

### 将摄像机绑定在物体的合适位置并通过C#脚本读取水平垂直输入

```
x += Input.GetAxis("Mouse X") * 0.2f;
y -= Input.GetAxis("Mouse Y") * 0.2f;
```
Input.GetAxis需要的字符串在Edit->Project Settings->Input中。

### 通过代码处理摄像机的角度与位置旋转

##### 关于欧拉角：

莱昂哈德•欧拉用欧拉角来描述刚体在三维欧几里得空间的取向。对于任何参考系，一个刚体的取向，是依照顺序，从这参考系，做三个欧拉角的旋转而设定的。所以，刚体的取向可以用三个基本旋转矩阵来决定。换句话说，任何关于刚体旋转的旋转矩阵是由三个基本旋转矩阵复合而成的。 

对于在三维空间里的一个参考系，任何坐标系的取向，都可以用三个欧拉角来表现。参考系又称为实验室参考系，是静止不动的。而坐标系则固定于刚体，随着刚体的旋转而旋转。

[欧拉角 - 维基百科](https://zh.wikipedia.org/wiki/%E6%AC%A7%E6%8B%89%E8%A7%92)

[欧拉角 - 百度百科](https://baike.baidu.com/item/%E6%AC%A7%E6%8B%89%E8%A7%92/1626212)

##### 代码具体实现：

```
x = cam.transform.eulerAngles.y;
y = cam.transform.eulerAngles.x;//获取当前的欧拉角
```

```
x += Input.GetAxis("Mouse X") * 0.2f;
y -= Input.GetAxis("Mouse Y") * 0.2f;//得到被改变的欧拉角
```

```
Quaternion rotate = Quaternion.Euler(y, x, 0);//得到相应的旋转四元数
```

```
cam.transform.rotation = rotate;//赋值给摄像机的transfrom
```


示例代码：
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class cameraContorl : MonoBehaviour {

    public GameObject cam;
    public float distance;

    void Update()
    {
        float x, y;

        x = cam.transform.eulerAngles.y;
        y = cam.transform.eulerAngles.x;

        distance -= Input.GetAxis("Mouse ScrollWheel");
        x += Input.GetAxis("Mouse X") * 0.2f;
        y -= Input.GetAxis("Mouse Y") * 0.2f;

        Quaternion rotate = Quaternion.Euler(y, x, 0);
        Vector3 positi = rotate * new Vector3(0.0f, 0.0f, -distance) + transform.position;

        cam.transform.rotation = rotate;
        cam.transform.position = positi;
       
    }
}
```

## 第三次技术分享

#### unity动画的简单介绍

实际上，unity动画部分十分复杂，并不太可能在短时间内讲清楚。这次介绍主要是对unity动画基本概念的介绍。

##### Animation：
unity动画的制作工具。是unity的旧版动画系统，一般来说3d模型的动画都应当由美术制作。

##### Animator：
一般来讲，在最近的unity版本中，动画都通过Animator实现。Animator模块可以编辑.controller文件，而.controller用于控制动画以及当前的状态。
###### Animator中的状态机可以可视化的调节动画的播放等。它提供了动画状态之间的切换功能、自带动画融合、能编辑动画播放的逻辑顺序、能设置随机播放、能设置行为树。

#### unity实现射击

说到unity的射击，就不得不提射线（Ray）。射线是unity预设的类。射线为一个从起点到终点，以特定方向发射的射线。
> A ray is an infinite line starting at origin and going in some direction.

发射射线和射线本身一样易于理解。实际上，让物体向前方发射射线只需要

```
Ray ray = new Ray(transform.position, transform.forward);
//要确保你的物体没有碰撞盒或者rigidbody，否则ray会停止
```
检测射线有没有接触物体，要使用RaycastHit类。
示例代码如下：

```
if (Input.GetMouseButtonDown(0))
        {
            RaycastHit hit;
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);

            if (Physics.Raycast(ray, out hit))
            {
                if (hit.collider != null)
                {
                    hit.collider.enabled = false;
                }
            }
        }
```
上面的代码包含了发射一个射线以及检测射线碰撞，射线碰撞到碰撞盒后将碰撞盒隐藏。
#### 粒子系统(Particle System)
可以用它做出相当棒的效果，比如雨雪等。（如何制作雨待补充，当然其实可以百度）
#### Line Renderer
通过代码操控实现画线。