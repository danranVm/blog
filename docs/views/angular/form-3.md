---
title: 使用指令增强 - Angular Form 三连(三)
date: 2019-12-11
prev: ./form-2
categories:
  - Angular
tags:
  - Angular
  - Form
---

::: tip
在上篇文章《表单源码解读》中，我们看到最多的词汇之一就是：指令(Directive)。  
Angular 使用指令让原生表单 DOM 有了一系列能力，那么我们是否也可以依葫芦画瓢去增强我们表单组件的体验呢？
:::

<!-- more -->

## 需求来源

在我们实际的开发场景中，对于表单，必不可少的就是验证和错误提示，我们往往需要：

```html
<!-- https://angular.io/guide/form-validation#template-driven-validation -->
<input id="name" class="form-control" formControlName="name" required />

<div *ngIf="name.invalid && (name.dirty || name.touched)" class="alert alert-danger">
  <div *ngIf="name.errors.required">
    Name is required.
  </div>
  <div *ngIf="name.errors.minlength">
    Name must be at least 4 characters long.
  </div>
  <div *ngIf="name.errors.forbiddenName">
    Name cannot be Bob.
  </div>
</div>
```

oh,shit! 这可真是这个操蛋的事情！  
不过一般我们都不会这么原始的去使用，通常我们会使用一套组件库，以[Nz-Zorro-Antd](https://ng.ant.design/docs/introduce/zh) 为例，`zorro` 在 `8.x` 的版本中，`nzFormControl` 组件支持了 `nzErrorTip` 属性。每次表单验证错误的时候，自动显示错误提示信息，这是一个很棒的功能！我们现在可以这么干了。

```html
<nz-form-item>
  <nz-form-control nzErrorTip="Input is required">
    <input nz-input formControlName="required" required />
  </nz-form-control>
</nz-form-item>
<nz-form-item>
  <nz-form-control nzErrorTip="MaxLength is 6">
    <input nz-input formControlName="maxlength" maxlength="6" />
  </nz-form-control>
</nz-form-item>
```

WOW! 是不是很酷！  
等等，你会发现这个例子和第一个例子有些区别，第一个例子是同时有多个验证规则的。那么，你也只能。。。

```html
<nz-form-item>
  <nz-form-control [nzErrorTip]="combineTpl">
    <input nz-input formControlName="name" />
    <ng-template #combineTpl let-control>
      <ng-container *ngIf="control.hasError('maxlength')">MaxLength is 12</ng-container>
      <ng-container *ngIf="control.hasError('minlength')">MinLength is 6</ng-container>
      <ng-container *ngIf="control.hasError('required')">Input is required</ng-container>
    </ng-template>
  </nz-form-control>
</nz-form-item>
```

oh~no! 这还不够酷！

## 痛点所在

通过上面的例子，我们可以发现目前 `nzErrorTip` 还遗留的两个痛点。

- 每个用 `nzFormControl` 的地方，都要写上一句 `Input is required` 又或者是 `Email is not valid`.
- 处理多个验证规则的体验依旧糟糕。

另外还有一个小小的"安全隐患"，错误提示都是写在 `template` 中，假如有一天我们的产品经理厌倦了 `Email is not valid` , 要求你把所有这个提示改成 `The input is not valid E-mail!`。 这个时候，你就只能摸一把眼泪，心里骂一句："\*\*\*\*", 然后的默默的打开 `VS CODE`, 带着沉重的心情按下 `Command Shift H` ,然后全局替换吧，如果还有队友写得不规范的话...

## 期望的样子

```html
<nz-form-item>
  <nz-form-control autoErrorTip>
    <input nz-input formControlName="mix" />
  </nz-form-control>
</nz-form-item>
```

我只需要打一个响指(`AutoErrorTipDirective`)，然后不管我是 `required` 还是 `minlength` 又或是 `maxlength`, 通通给我显示出对应的错误提示。

## 具体实现

知道问题所在，以及期望的样子。那么，就是现在，让我们把它变得更酷一些！

已经实现了一个 `demo`: [stackblitz](https://stackblitz.com/edit/ng-zorro-antd-auto-error-tip?file=src%2Fapp%2Fauto-error-tip.directive.ts)

```ts
// 核心代码如下
this.ngControl.statusChanges.pipe(filter(status => status === 'INVALID')).subscribe(() => {
  const errors = this.ngControl.errors || {}
  Object.entries(errors).some(([key, value]) => {
    const errorTip = value[this.errorTipKey]
    if (!!errorTip) {
      this.nzFormControl.nzErrorTip = errorTip
    } else if (this.errorTipMap[key]) {
      this.nzFormControl.nzErrorTip = this.errorTipMap[key]
    }
    return !!errorTip
  })
  this.cdr.markForCheck()
})
```

### 实现思路

- 由于 `autoErrorTip` 指令是挂在 `NzFormControlComponent` 组件上的，所以很容易就能拿到 `NzFormControlComponent` 和 `NgControl` 的实例。
- 订阅 `NgControl.statusChanges` 事件，然后过滤出我们想要的 `INVALID` 状态。
- 拿到当前状态下 `NgControl` 的错误信息对象 `errors`, 遍历它，根据我们约定好的 `errorTipKey` 找到对应的错误提示 `errorTip`.
- 赋值给 `nzFormControl` 的 `nzErrorTip`.

### 如何使用

需要预先约定好两个数据： `errorTipKey` 和 `errorTipMap`， 优先级如下：

- 通过 `@Input` 的方式设置 `errorTipKey` 和 `errorTipMap`
- 通过依赖注入(如果在 `zorro` 内实现，可以换成全局配置)的方式设置 `errorTipKey` 和 `errorTipMap`
- 给定默认的一个 `errorTipKey`

对于 `Angular` 官方的 `Validators`, 我们有两种处理方式：

- 在 `errorTipMap` 中根据错误类型声明对应的 `errorTip` ，示例:

  ```ts
  const autoErrorTipMap: Record<string, string> = {
    required: 'Input is required',
    email: 'The input is not valid email',
  }
  // 注入到 app.module
  providers: [{ provide: AUTO_ERROR_TIP_MAP, useValue: autoErrorTip }]
  ```

- 自定义一个 `MyValidators extends Validators` ，示例:

  ```ts
  // 约定使用 errorTip 约束所有验证器的实现
  export type MyErrorsOptions = { errorTip: string } & Record<string, any>
  export type MyValidationErrors = Record<string, MyErrorsOptions>

  export class MyValidators extends Validators {
    static minLength(minLength: number): ValidatorFn {
      return (control: AbstractControl): MyValidationErrors | null => {
        if (Validators.minLength(minLength)(control) === null) {
          return null
        }
        return { minlength: { errorTip: `MinLength is ${minLength}` } }
      }
    }

    // 除了官方的验证器，当然我们还可以自定义一些其他的验证器。
    static mobile(control: AbstractControl): MyValidationErrors | null {
      const value = control.value
      if (isEmptyInputValue(value)) {
        return null
      }
      return isMobile(value) ? null : { mobile: { errorTip: 'Mobile phone number is not valid' } }
    }
  }
  ```

## 思考

你也许已经注意到了，我所有的示例都是 `ReactiveForm`, 那么是不是就不支持 `TemplateForm` 呢？当然不是啦。  
通常情况下，`ReactiveForm` 是更被推荐的使用方式，我也更推荐你使用它搭配 `autoErrorTip`，另外它实现起来也更加容易，所以我才用它作为示例。如果你需要支持 `TemplateForm` 的话，就跟自定义 `Validators` 一样，你需要实现根据[官方的`ValidatorDirectives`](https://github.com/angular/angular/blob/master/packages/forms/src/directives/validators.ts#L160)实现一套 `MyValidatorDirectives`.

那么，是不是只有表单组件可以这样子去增强呢？当然不是啦。  
这是一种实现思路，适用于各种组件，期待你跟我分享其他场景下的方案。

## 更多

- [自定义表单组件 - Angular Form 三连(一)](./form-1.md)
- [表单源码解读 - Angular Form 三连(二)](./form-2.md)
