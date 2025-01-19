---
title: AI 代码生成产品体验
date: 2025-01-19 07:17:46
categories:
tags:
---

##### AUTHOR: [Locez](https://locez.com)

##### VERSION: 1

### 背景

---

AI 代码生成器是一个非常有用的工具，它可以帮助程序员快速生成代码，提高开发效率。现在市面上有很多 AI 代码生成器，例如 ChatGPT、GitHub Copilot、Cursor 等。这些 AI 代码生成器都宣传自己的代码生成能力非常强，因此今天我们就来看看这些 AI 代码生成器的用户体验。

由于笔者比较熟悉 `C++` 和 `Python`，本文主要以这两种语言进行评测。另外就是该评测主要由评测人员的水平决定（也就是我的主观意识），因此请不要将评测结果作为 AI 代码生成器的唯一依据。我只能按照我对代码标准的理解来评价 AI 代码生成的表现。

### 评测产品

---

由于有大量的 AI 产品，以及我个人的精力有限，有些需要 API token，我也没有足够的精力去测试。因此，我只测试了以下几个 AI 代码生成器：

- Cursor（claude-3.5-sonnet）
- DeepSeek
- Marscode

后续有时间会增加其它产品的体验

### 评测维度

---

我们来看看 AI 代码生成器的评测维度，包括以下几个方面：

- 功能的正确性
- 代码理解和解释能力
- 代码生成能力
- 交互方式

评分以 `5` 分为满分标准，打分涉及主观，请酌情谅解。

### Python

---

#### Question 1

---

```
请生成一个用户注册功能的代码，要求如下：
1. 使用 Python 和 Flask 框架。
2. 用户信息包括用户名、密码、邮箱。
3. 密码需要加密存储。
4. 返回注册成功的 JSON 响应。
```

| 产品     | 结果                                              | 得分 |
| -------- | ------------------------------------------------- | ---- |
| cursor   | 功能完整，且处理了异常，但是代码结构不如 deepseek | 4.8  |
| deepseek | 功能完整，但未处理异常                            | 4.5  |
| marscode | 生成未涉及到数据库配置和存储，不足以直接生产应用  | 4    |

#### Question 2

---

```
请生成一个用户登录功能的代码，要求如下：
1. 使用 Python 和 Flask 框架。
2. 验证用户名和密码。
3. 返回登录成功的 JSON 响应，包含 JWT Token。
```

| 产品     | 结果                                                 | 得分 |
| -------- | ---------------------------------------------------- | ---- |
| cursor   | 在追加生成中，生成功能完整，但是未 import 对应模块   | 4.5  |
| deepseek | 功在追加生成中，生成功能完整，但是未 import 对应模块 | 4.5  |
| marscode | 直接触发了 bug                                       | 0    |

附赠 marscode 的 bug 代码，一直在循环生成 8X8Y，没搞明白为什么

```python
# 模拟数据库存储用户信息
users = [
    {
        'username': 'testuser',
        'password': '$2b$12$I9T8W6v2Z0W0n8Y2g6g7uO8X8Y8X8Y8X8Y8X8Y8X8Y8X....
```

### 总结

---

写到此处，发现要详尽的列出测试过程，还有一些使用方式的问题比较困难，因此接下来直接给出一些评分项

#### 评测结果

---

| 产品     | 代码生成能力 | 生成注释 | 代码解释能力 | 单测生成能力 | Bug 查找能力 | 总分 |
| -------- | ------------ | -------- | ------------ | ------------ | ------------ | ---- |
| cursor   | 4.8          | 4.9      | 4.5          | 4.0          | 4.5          | 22.7 |
| deepseek | 4.5          | 4.3      | 4.3          | 4.5          | 3.5          | 21.1 |
| marscode | 3.5          | 4.2      | 3.0          | 4.6          | 3.0          | 18.3 |

`cursor` 是我第一次体验，不太清楚是内部优化了还是 `claude-3.5-sonnet` 模型的优势（在此之前我未用过），整体表现是非常不错的。操作上总体和 `deepseek` 很接近，但是 `cursor` 更加保守一些，生成的代码健壮性会更强。`deepseek` 是以 `continue` 插件接入的，有些使用体验细节略微差了一点

`marscode` 则在代码方便能力稍弱，但是它提供了一个很方便的插件，有一些操作直接点击插件上显示的按钮即可实现。生成单测能力来看是体验非常好的。

### 附录

---

**NOTE**: 其中的 bug 查找能力，来源于我之前写的一个 `c++` BUG，这个 BUG 导致的根本原因是 remove_if 是 copy 覆盖实现的，因此线程在这个移动的过程中就已经出现了异常，AI 时常会觉得删除最后的元素就完事了，这三个都没能识别出真正的问题，其中 `deepseek` 和 `marscode` 甚至没能修复，但是 Cursor 的保守策略使得他转用了 `find_if` 来实现相似的功能，相当于修复了该程序，源程序如下：

```cpp

#include <thread>
#include <mutex>
#include <vector>
#include <iostream>
#include <atomic>
#include <algorithm>

int main(){

  std::vector<std::pair< std::thread::id,std::thread>> threads;
  std::atomic_bool running;
  std::mutex mutex;
  running = true;

  auto worker = [&](){
    std::this_thread::sleep_for(std::chrono::seconds(1));
    auto id = std::this_thread::get_id();

    std::lock_guard<std::mutex> lock(mutex);
    auto it = std::remove_if(threads.begin(), threads.end(), [&](const auto& pair){    return pair.first == id;});
    it->second.detach();
    return;
  };

  {

    std::lock_guard<std::mutex> lock(mutex);
    for (int i =0 ; i< 5; i++) {
      std::thread thread = std::thread(worker);
      threads.emplace_back(thread.get_id(), std::move(thread));
    }
  }
  std::this_thread::sleep_for(std::chrono::seconds(20000));

  return 0;

}

```
