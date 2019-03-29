# 在 Angular 中引入 Jest 进行单元测试

## 为什么要从 Karma 迁移到 Jest

### 用 Karma 在项目中遇到了坑

最近新换了一个项目，去的时候项目已经做了两个月了，因为前期赶功能，没有对单元测试做要求，CI/CD 的时候也没有强制跑单元测试。所以虽然有用 Angular CLI 自动生成的测试文件，但是基本上都是测试不通过。
项目做久了，人员变动多，新来的成员对之前的业务逻辑不清不楚，稍不注意就会破坏之前的功能；业务复杂了，随便增加或者修改一点点功能都可能引起不易被察觉的 BUG。作为一个敬业的开发，不上单元测试怎么行。所以，就有了一个修复已有单元测试的任务。
修复已有测试文件的思路很简单：写个 TestingModule 把常用的依赖 mock 掉，再引入到需要的文件中就行了；不常用的依赖，在各自的文件中 mock 掉就好了。
然而实际操作起来的时候，Karma 早早挖好坑等这了。有些测试文件单跑没有问题，整体跑得时候就报错，测试结果及其不稳定；karma 的报错信息又特别难读懂，很多时候根本定位不到到底是哪里出了问题。再加上 Karma 需要先把 Angular 应用编译之后再在浏览器中跑测试，整体时间也比较慢，修复的过程一直处于抓狂的边缘。
整体测试跑起来的时候难以定位测试出错的定位，怎么办呢，那就让跑整个测试的时候各个文件之间也没有依赖可以单独跑好了，所以就想到了 Jest。实践证明，在 Angular 中， Jest 大法也非常好使。

### Karma 和 Jest 的对比

前面也说过了，在修复测试的过程中，karma 遇到了各种各样的问题。归结起来大概就是：
- Karma 需要先把 Angular 应用整体编译之后再在浏览器中跑测试，跑测试的时间比较长；
- Karma 测试结果不稳定（很可能是因为异步操作引起的），单个文件和整体测试时的测试结果不一致；
- 报错信息模糊不清，无法定位问题。特别是在有大量测试需要修复的情况下，难以定位问题的根本原因。
那么对比而言，Jest 在上面这些方面都有很好的表现：
- 不需要整体编译，可以单文件测试
- 测试结果稳定
- 报错清楚，易于定位问题

## 迁移

第一步，你需要相关依赖包：

```
npm install --save-dev jest jest-preset-angular @types/jest
````
其中：
- jest – Jest 测试框架
- jest-preset-angular – jest 对于 angular 的一些通用的预设置
- @types/jest – Jest 的 typings

第二步，你需要在 package.json 中对 Jest 进行配置：
```
"jest": {
  "preset": "jest-preset-angular",
  "setupFilesAfterEnv": ["<rootDir>/src/setupJest.ts"]
}
```
其中，`preset` 声明了预设，`setupFilesAfterEnv` 配置了 Jest setup 文件的地址，可以包含多个文件，这里设置的是项目根目录下的 `src/setupJest.ts`。

第三步，在 src 目录下创建上一步中设置的 setup 文件 `setupJest.ts`
```
import 'jest-preset-angular'; // jest 对于 angular 的预配置
import './jestGlobalMocks'; // jest 全局的 mock
```

第四步，在 src 目录下创建 `jestGlobalMocks.ts` 文件，并加入相关的全局的 mock，以下是一个例子：
```
const mock = () => {
  let storage = {};
  return {
    getItem: key => key in storage ? storage[key] : null,
    setItem: (key, value) => storage[key] = value || '',
    removeItem: key => delete storage[key],
    clear: () => storage = {},
  };
};

Object.defineProperty(window, 'localStorage', {value: mock()});
Object.defineProperty(window, 'sessionStorage', {value: mock()});
Object.defineProperty(window, 'getComputedStyle', {
  value: () => ['-webkit-appearance']
});
```
可以看到这个例子中 mock 了 window 上的对象，这是因为 jsdom 并没有实现所有的 window 上的对象和方法，所以有时我们需要自己给 window 打个补丁。
在这里 mock `localStorage` 是可选的，如果我们在代码中并没有使用。但是 mock `getComputedStyle` 是必须的，因为 Angular 会检查它在哪个浏览器中执行。如果没有 mock `getComputedStyle`，我们的测试代码将无法执行。



# UseJestInAngular

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.3.5.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).
