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
