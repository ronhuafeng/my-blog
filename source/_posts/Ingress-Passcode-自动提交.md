---
title: Ingress_Passcode_自动提交
date: 2017-01-18 13:10:10
categories:
  - 技术记录
tags:
---

## 什么是 Passcode？

根据 [Ingress 中文游戏指南](https://www.gitbook.com/book/hz-ingress/ingress-tutorial/) 的讲解，Passcode 是 Ingress 提供的一种奖励。

> 使用 Investigation Board 来了解 Ingress 的最新剧情并寻求对 Niantic Project, NIA, XM, Shapers 以及各阵营不同问题的答案。 可以使用隐藏在这些报告之中的 Passcode 来兑换道具（Resonator，XMP 等），XM 或者 AP。

> 另外，一些官方活动会向到场者赠送包含 Passcode 的卡片，特定的官方周边也会赠送此类卡片。卡片上的 Passcode 一般能够兑换活动徽章。

注：在很多社交网站上可以找到有效的 Passcode 

在 iOS 平台的 Ingress 客户端上 Passcode 兑换功能不可用，因此只能使用 Intel Map 来兑换。

## 在 Intel Map 上兑换

1. 在右上方点击 Passcode
2. 输入你的 passcode，然后点击 SUBMIT 兑换

## 自动化 Intel Map 兑换过程

可以通过浏览器的调试页面，自动将搜寻到的 passcode 提交，进行物品兑换。

1. 表单提交
 
 ```javascript
 document.getElementById("redeem_reward_form").submit();
 ```
 不能正确提交表单失败的原因是这个表单的 `input` 元素名称就是 `submit`，需要重命名。

 ```javascript
 var field = document.getElementById("chicken");
 field.id = "horse";  // using element properties
 field.setAttribute("name", "horse");  // using .setAttribute() method
 document.getElementById("chicken").setAttribte('name', 'horse');
 document.getElementById("redeem_reward_form").submit.setAttribute('id', 'submit_code')
 ```
 这些都不太管用，应该是 form 之前的 submit 属性已经被设置了，改了名字也不能绑定 submit 方法了。:|
 
 有效的代码：
 ```javascript
 document.getElementById("redeem_reward_form").submit.click()
 ```
2. 每隔一段时间自动提交一个 passcode
  
  ```javascript
  var codes = ['DEADDROP7DT73AM6'];   //Here put the number of times you want to auto submit
  var count = 0;
  (function submitPasscode(){
    if(count >= codes.length) return;
    document.getElementById("redeem_reward_form").passcode.value = codes[count];
    document.getElementById("redeem_reward_form").submit.click();
    count++;
    
    setTimeout(submitPasscode, 3000 + Math.floor((Math.random() * 500) + 1););   //Each second
  })(); 
  ```
3. 增加一个输入窗口，用来输入获得的 Passcodes
  
  Passcode 收集过来后一行一行整理好：
  ```
  ada3zc36qq9
  ada9yv83mp5
  algorithm9ek27ux3
  algorithm9gh35cj3
  artifact3ne73hh3
  bletchley9ob65ca4
  blue2xc26da2
  ```
  ```javascript
  function susubmitPasscodes()
  {
    var codes = document.getElementById('passcodesArea').value.replace( /\n/g, " " ).split(" ");
    var count = 0;
    console.log('Submit code count: ' + codes.length + '\n');
    function submitPasscode(){
      if(count >= codes.length) 
      {
        count=0;
        alert('All passcodes are processed.');
        return;
      }
      document.getElementById("redeem_reward_form").passcode.value = codes[count];
      console.log('Submit code: ' + codes[count] + '\n');
      document.getElementById("redeem_reward_form").submit.click();
      count++;
      
      setTimeout(submitPasscode, 25000 + Math.floor((Math.random() * 25000) + 1)); 
    }; 
    submitPasscode();
  };

  var passcodeBox = document.getElementById('header_passcode_box');

  
  var passcodesArea = document.createElement('textarea');
  passcodesArea.id = 'passcodesArea';
  passcodeBox.appendChild(passcodesArea);
  
  var batchSubmit = document.createElement("button");
  //<button onclick="myFunction()">Click me</button>
  batchSubmit.type = 'button';
  batchSubmit.name = 'batch-submit';
  batchSubmit.value = 'Submit';
  batchSubmit.onclick = susubmitPasscodes;
  batchSubmit.textContent = 'batchSubmit'
  passcodeBox.appendChild(batchSubmit);
  ```


