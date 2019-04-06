---
title: 不会测试的UI不是好前端
date: 2019-03-29
tags:
    - js
    - qa
---

**自 js 崛起，GitHub 上有越来越多的开源项目，而每个项目的根目录下必然会有 test，用来做项目的自测。js 已经入侵测试领域了。**

**本贴记录一些常用的测试库，和基本使用方法。**

## 一、单元测试

**使用 karma、phantomjs（无头浏览器） 进行单元测试（小函数、功能），并使用 karma-coverage 生成报表。**

**使用 npm 安装 karma，karma-coverage，jasmine-core，karma-jasmine，karma-phantomjs-launcher。然后调用 karma init 进行初始化，在根目录下得到 karma.conf.js 文件。配置如下：**

```js
{
    basePath: '',
    frameworks: ['jasmine'],
    files: [
      './tests/unit/*.js',
      './tests/unit/*.spec.js'
    ],
    exclude: [
    ],
    preprocessors: {
      './tests/unit/**/*.js': ['coverage']
    },
    reporters: ['progress', 'coverage'],
    coverageReporter: {
      type: 'html',
      dir: './docs/coverage/'
    },
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: false,
    browsers: ['PhantomJS'],
    singleRun: true,
    concurrency: Infinity
  }
```

**/tests/unit/index.js 文件**

```js
function add(num) {
    if (num === 1) {
        return 1;
    }
    
    return num + 1;
}
```

**/tests/unit/index.spec.js 文件**

```js
describe('加一函数测试', function () {
    it('加一函数是否可用', function () {
        // expect(add(1)).toBe(1);
        expect(add(2)).toBe(3);
    });
})
```

**运行 karma start 得到如下报表，其中得到测试结果以及提醒测试js中有部分代码没有覆盖到。**

![unit test reporters](../../../../img/web_qa/unit_test_reporters.png)

## 二、性能测试

## 三、安全测试

## 四、功能测试

### 模拟用户操作

**使用 selenium-webdriver 模拟用户操作。**

**用 npm 安装 selenium-webdriver，并在 selenium-webdriver 的npm 网站上下载相关浏览器的 driver，解压在根目录下。**

**以下是调用 selenium-webdriver 的官方例子，node运行如下js，测试结果在终端输出。**

```js
const { Builder, By, Key, until } = require('selenium-webdriver');

(async function example() {
    let driver = await new Builder().forBrowser('chrome').build();
    try {
        await driver.get('http://www.baidu.com');
        await driver.findElement(By.name('wd')).sendKeys('webdriver', Key.ENTER);
        await driver.wait(until.titleIs('webdriver_百度搜索'), 3000);
    } finally {
        await driver.quit();
    }
})();
```

![e2e test](../../../../img/web_qa/e2e_test.png)



### 后台服务测试

**使用 mocha 进行后台接口数据测试。**

**使用 npm 安装 express（node服务库），mocha，mochawesome（生成报表），supertest（代理服务）**

**app.js 文件**

``` js
const express = require('express');
const app = express();

app.get('/data', (req, res) => {
    res.send({
        name: 'jqx'
    })
})

const server = app.listen(3000, () => {
    console.log('server start at 3000');
});

module.exports = app;
```

**app.spec.js 文件**

```js
const superagent = require('supertest');
const app = require('./app.js');

function req() {
    return superagent(app.listen());
}

describe('node 接口测试', () => {
    it('data 接口测试', done => {
        req()
            .get('/data')
            .expect(200)
            .expect('Content-Type', /json/)
            .end((err, res) => {
                if (err) {
                    done(err);
                }

                if (res.body.name === 'jqx') {
                    done();
                } else {
                    done(new Error('名字错误'));
                }
            })
    })
})
```

**根目录下创建 mochaRunner.js 文件**

```js
const Mocha = require('mocha');

let mocha = new Mocha({
    reporter: 'mochawesome',
    reporterOptions: {
        reportDir: './docs/customReportFilename',
        // quiet: true
    }
});

mocha.addFile('app.spec.js');
mocha.run(() => {
    process.exit();
})
```

**node 运行 mochaRunner.js，并生成报表**

![mocha test](../../../../img/web_qa/mocah_test.png)

## 五、UI测试

**使用 backstopjs 进行 ui 测试。**

**使用 npm 安装 backstopjs。安装完后运行 backstop init 命令初始化。**

**配置根目录下的 backstop.json**

``` js
{
  "id": "taobao",
  "viewports": [
    { // 测试网页的尺寸
      "label": "phone",
      "width": 320,
      "height": 480
    },
    {
      "label": "tablet",
      "width": 1024,
      "height": 768
    }
  ],
  "onBeforeScript": "puppet/onBefore.js",
  "onReadyScript": "puppet/onReady.js",
  "scenarios": [
    {
      "label": "taobao index",
      "cookiePath": "backstop_data/engine_scripts/cookies.json", // 如果需要cookie则进行配置
      "url": "https://h5.m.taobao.com/?sprefer=sypc00", // 测试网页的地址
      "referenceUrl": "",
      "readyEvent": "",
      "readySelector": "",
      "delay": 0,
      "hideSelectors": [],
      "removeSelectors": [],
      "hoverSelector": "",
      "clickSelector": "",
      "postInteractionWait": 0,
      "selectors": [],
      "selectorExpansion": true,
      "expect": 0,
      "misMatchThreshold" : 0.1,
      "requireSameDimensions": true
    }
  ],
  "paths": {
    "bitmaps_reference": "backstop_data/bitmaps_reference", // ui 原图路径
    "bitmaps_test": "backstop_data/bitmaps_test",
    "engine_scripts": "backstop_data/engine_scripts",
    "html_report": "./docs/backstop_data/html_report", // 测试报表生成地址
    "ci_report": "./docs/backstop_data/ci_report"
  },
  "report": ["browser"],
  "engine": "puppeteer",
  "engineOptions": {
    "args": ["--no-sandbox"]
  },
  "asyncCaptureLimit": 5,
  "asyncCompareLimit": 50,
  "debug": false,
  "debugWindow": false
}
```

**运行 backstop test 进行测试。生成报表中，像素点不同之处都会标红，如下**

![backstop test](../../../../img/web_qa/backstop_test.png)