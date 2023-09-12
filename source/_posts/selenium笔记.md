---
title: selenium笔记
date: 2023-08-09 18:48:13
updated: 2023-08-09 18:48:13
toc: true
cover:
thumbnail:
categories: 笔记
tags: 
    - Pyhton
    - Selenium
---


去年学习selenium模块时做的的笔记，当时看完[网上教程](https://www.byhy.net/tut/auto/selenium/01/)后写了一个简单的项目。
<!-- more -->

## 全局操作

### 防止日志写屏

```python
## 加上参数，禁止 chromedriver 日志写屏
options = webdriver.ChromeOptions()
options.add_experimental_option('excludeSwitches', ['enable-logging'])

## 创建 WebDriver 对象，指明使用chrome浏览器驱动
wd = webdriver.Chrome(options=options)  ## 这里指定 options 参数
```

### 设置等待时间

Selenium 的 Webdriver 对象有个方法叫 implicitly_wait ，可以称之为**隐式等待**，或者**全局等待**。
该方法接受一个参数，用来指定最大等待时长。

```python
wd.implicitly_wait(10)
```


## 操作元素

### [操作元素的基本方法](https://www.byhy.net/tut/auto/selenium/03/)

1. 点击元素：click()
2. 输入框
   * 清除（如有必要）：clear()
   * 输入：send_keys('str') 

### 获取元素信息

1. 文本内容：text
2. 属性：get_attribute('class')
3. HTML：element.get_attribute('outerHTML') 和 element.get_attribute('innerHTML')
4. 输入框文字：element.get_attribute('value')
5. 文本内容2：
   * 可见文本内容：element.get_attribute('innerText')
   * 所有内容：element.get_attribute('textContent')

#### CSS Selector

### tag, id, class

1. find_element(By.CSS_SELECTOR, CSS Selector参数) id, .class, ##id
2. find_element(By.ID, 'id')

### 子元素，后代元素

1. 直接子元素：元素1 > 元素2
2. 后代元素： 元素1 元素2

### 根据属性

1. find_element(By.CSS_SELECTOR, CSS Selector参数[attribute='str'])  可以不写标签参数和具体属性值
2. *= 包含
3. ^= 开头
4. $= 结尾
5. 多属性：CSS Selector参数\[attribute1='str1'][attribute2='str2']

### 验证CSS Selector

直接在浏览器开发者工具中验证

### [更高阶](https://www.byhy.net/tut/auto/selenium/css_2/)


## frame切换/窗口切换

### 切换到frame

1. wd.switch_to.frame(frame_reference) frame_reference 可以是 frame 元素的属性 name 或者 ID
2. wd.switch_to.frame(wd.find_element(By.TAG_NAME, "tag"))

切换回主HTML：wd.switch_to.default_content()


### 切换到新的窗口

1. wd.switch_to.window(handle)

```python
for handle in wd.window_handles:
    ## 先切换到该窗口
    wd.switch_to.window(handle)
    ## 得到该窗口的标题栏字符串，判断是不是我们要操作的那个窗口
    if 'target' in wd.title:
        ## 如果是，那么这时候WebDriver对象就是对应的该该窗口，正好，跳出循环，
        break
```

2. 保存当前窗口句柄
   wd.current_window_handle

## [选择框](https://www.byhy.net/tut/auto/selenium/skills_1/)

### radio

1. 当前选中元素：'input[checked="checked"]'
2. 选择元素：element.click()

### checkbox

1. 必须**先获取当前该复选框的状态** ，如果该选项已经勾选了，就不能再点击。否则反而会取消选择
2. 先把 已经选中的选项全部点击一下，确保都是未选状态再点击目标

### select

1. 根据选项的 value属性值: .select_by_value('value')
2. 根据选项的 次序 （从1开始）: .select_by_index()
3. 根据选项的 可见文本：.select_by_visible_text()
4. deselect 可去除选中元素
5. deselect_all 去除选中所有元素




## 实战技巧

### [更多动作：ActionChains类](https://www.byhy.net/tut/auto/selenium/skills_2/)

```python
from selenium.webdriver.common.action_chains import ActionChains
```

### 直接执行javascript

### 冻结页面

在 开发者工具栏 console 里面执行如下js代码

```javascript
setTimeout(function(){debugger}, 5000)
```

表示在 5000毫秒后，执行 debugger 命令
执行该命令会 浏览器会进入debug状态。debug状态有个特性，界面被冻住，不管我们怎么点击界面都不会触发事件。

### 弹出对话框

1. Alert
   如果我们不去点击它，页面的其它元素是不能操作的。
   模拟用户点击: driver.switch_to.alert.accept()

2. Confirm
   OK: driver.switch_to.alert.accept()
   Cancle: driver.switch_to.alert.dismiss()

3. Prompt
   driver.switch_to.alert.send_keys()

**有些弹窗并非浏览器的alert 窗口，而是 html元素 ，这种对话框，只需要通过之前介绍的选择器选中并进行相应的操作就可以了。**


### 其他技巧

1. 窗口大小

   * 获取：driver.get_window_size()
   * 改变：driver.set_window_size(x, y)

2. 窗口标题
   driver.title

3. URL地址
   driver.current_url

4. 截屏
   driver.get_screenshot_as_file('1.png')

5. 手机模式

   ```python
   from selenium import webdriver
   mobile_emulation = { "deviceName": "Nexus 5" }
   chrome_options = webdriver.ChromeOptions()
   chrome_options.add_experimental_option("mobileEmulation", mobile_emulation)
   driver = webdriver.Chrome( desired_capabilities = chrome_options.to_capabilities())
   driver.get('http://www.baidu.com')
   input()
   driver.quit()
   ```

6. 上传文件
   send_keys()


## [Xpath选择器](https://www.byhy.net/tut/auto/selenium/xpath_1/)

[attribute2='str2']: 
