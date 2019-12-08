---
title: 使用指令增强 - Angular Form 三连(三)
date: 2019-12-09
prev: ./form-2
categories:
  - Angular
tags:
  - Angular
  - Form
---

::: tip
在上篇文章《表单源码解读》中，我们看到最多的词汇之一就是：指令(Directive)。
:::

<!-- more -->

`zorro` 在 `8.x` 的版本中，`nzFormControl` 支持了 `nzErrorTip` 属性。每次表单验证错误的时候，自动显示错误提示信息，这是一个很棒的功能，但是它还可以变得更加方便！  
那就是 `autoErrorTip` ,它主要解决了目前 `nzErrorTip` 还遗留的两个痛点。

- 重复定义错误提示：也就是我每个用 `nzFormControl` 的地方，都要写上一句 `Input is required` 又或者是 `Email is not valid`.
- 处理多种错误类型的时候不是很方便，示例：

  ```html
  <nz-form-control [nzSpan]="12" nzHasFeedback [nzErrorTip]="emailErrorTpl">
    <input nz-input formControlName="email" placeholder="email" type="email" />
    <ng-template #emailErrorTpl let-control>
      <ng-container *ngIf="control.hasError('email')">
        The input is not valid E-mail!
      </ng-container>
      <ng-container *ngIf="control.hasError('required')">
        Please input your E-mail!
      </ng-container>
    </ng-template>
  </nz-form-control>
  ```

另外它还可以带来一个额外的好处，就是标准化错误提示，可以让我们全局的错误提示保持统一，也方便替换，否则你设想一下：

- 假如有一天我们的产品经理厌倦了 `Email is not valid` , 要求你把所有这个提示改成 `The input is not valid E-mail!`。 这个时候，你就只能全局去替换了。

```html
<form nz-form [formGroup]="formGroup">
  <nz-form-item>
    <nz-form-control autoErrorTip>
      <input type="text" nz-input formControlName="userName" required />
    </nz-form-control>
  </nz-form-item>
</form>
```

已经实现了一个 `demo`: https://stackblitz.com/edit/ng-zorro-antd-auto-error-tip?file=src%2Fapp%2Fauto-error-tip.directive.ts

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

实现思路其实挺简单的，就是比根据 `NgControl` 的错误类型，使用约定好的 `errorTipKey` 找到对应的错误提示 `errorTip` ，然后赋值给 `nzFormControl` 的 `nzErrorTip`:

对于使用者，需要预先约定好两个数据： `errorTipKey` 和 `errorTipMap`， 优先级如下：

- 通过 `@Input` 的方式设置 `errorTipKey` 和 `errorTipMap`
- 通过依赖注入(如果在 `zorro` 内实现，可以换成全局配置)的方式设置 `errorTipKey` 和 `errorTipMap`
- 给定默认的一个 `errorTipKey` 和一组 `errorTipMap`

对于 `Angular` 官方的 `Validators`, 我们有两种处理方式：

- 在 `errorTipMap` 中根据错误类型声明对应的 `errorTip` ，示例:

  ```ts
  const autoErrorTipMap: Record<string, string> = { email: 'The input is not valid email' }
  providers: [{ provide: AUTO_ERROR_TIP_MAP, useValue: autoErrorTip }]
  ```

- 自定义一个 `MyValidators extends Validators` ，示例:

  ```ts
  // 约定使用 `errorTip` 约束所有验证器的实现
  export type MyErrorsOptions = { errorTip: string } & Record<string, any>

  export type MyValidationErrors = Record<string, MyErrorsOptions>

  export class MyValidators extends Validators {
    static maxLength(maxLength: number): ValidatorFn {
      return (control: AbstractControl): MyValidationErrors | null => {
        const length: number = control.value ? control.value.length : 0
        return length > maxLength
          ? {
              maxlength: {
                errorTip: `MaxLength is ${maxLength}`,
                requiredLength: maxLength,
                actualLength: length,
              },
            }
          : null
      }
    }

    static mobile(control: AbstractControl): MyValidationErrors | null {
      const value = control.value

      if (isEmptyInputValue(value)) {
        return null
      }

      return isMobile(value)
        ? null
        : { mobile: { errorTip: 'Mobile phone number is not valid', actual: value } }
    }
  }
  ```

## 更多

- [自定义表单组件 - Angular Form 三连(一)](./form-1.md)
- [表单源码解读 - Angular Form 三连(二)](./form-2.md)
