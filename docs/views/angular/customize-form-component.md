---
title: Angular 自定义表单组件
date: 2019-08-04
next: ./source-code-of-the-form
categories:
  - Angular
tags:
  - Angular
  - Form
---

::: tip
首先我们应该知道在 Angular 中，表单组件应该有哪些特性：

- view <==> model 的双向绑定
- 支持 template-form & reactive-form
- 支持表单验证
- 组件状态的控制，用于 CSS 类映射到组件所在的元素

现在，我们来实现它们。
:::

<!-- more -->

## view <==> model 的双向绑定

首先，我想说的是，Angular 中的双向绑定只不过是一个输入和输出的语法糖。

### 定义一个 `MyInput` 组件

```ts
@Component({
  selector: 'my-input',
  template: `
    <input [value]="value" (input)="valueChange.emit($event.target.value)" />
  `,
})
export class MyInputComponent {
  @Input() value: string = '0'
  @Output() valueChange: EventEmitter<string> = new EventEmitter()
}
```

`MyInput` 组件有一个输入属性 `value`，有一个输出属性 `valueChange`。
看到这里，也许你已经知道了双向绑定的是怎么一回事了，它只是使用 `[()]` 简化了输入 `xxx`和输出 `xxxChange` 的绑定语法糖而已。

### 在父组件中使用

可以看到，两种使用方式都效果完全一致。

```ts
@Component({
  selector: 'app-root',
  template: `
    <my-input [(value)]="value"></my-input>
    <my-input [value]="value" (valueChange)="value = $event"></my-input>
  `,
})
export class AppComponent {
  value = '100'

  constructor() {
    setInterval(() => console.log(this.value), 1000)
  }
}
```

当你输入新的值时，观察控制台的输出，就会是你输入后的新值。

## 支持 template-form & reactive-form

在 Angular 中，表单组件又分为模板驱动表单和响应式表单，我们的 `MyInput2` 组件需要都支持它们的使用方式。

### 实现方式

其实很简单，我们只需要让 `MyInput2` 提供 `NG_VALUE_ACCESSOR` 令牌并实现一个接口 `ControlValueAccessor` 。

#### implements & providers

```ts
const accessorProvide = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => MyInput2Component),
  multi: true,
}

@Component({
  selector: 'my-input2',
  template: ``, // 需要调整
  providers: [accessorProvide],
})
export class MyInput2Component implements ControlValueAccessor {}
```

然后编辑器就回提示我们需要去实现 `ControlValueAccessor` 中的方法，使用 vscode 的快速修复快捷键，会自动为我们创建所需要实现的方法。

```ts
writeValue(obj: any): void // 当 model 发生改变时，触发该方法。
registerOnChange(fn: any): void // 注册 onChange 事件。
registerOnTouched(fn: any): void // 注册 onTouched 事件
setDisabledState?(isDisabled: boolean): void // 设置组件的禁用状态，可选实现。
```

注：关于 `NG_VALUE_ACCESSOR` 令牌 ，下篇文章[Angular 表单的源码解读](./source-code-of-the-form.md)将为你详细介绍。

#### 实现 `ControlValueAccessor` 的方法

```ts
export class MyInput2Component implements ControlValueAccessor {
  value: string = '0'
  isDisabled = false

  onChange = (value: string) => {}
  onTouched = () => {}

  writeValue(value: string): void {
    this.value = value
    // 如果你使用的是 OnPush 策略，别忘了下面一行代码
    // this.changeDetectorRef.markForCheck()
  }

  registerOnChange(fn: any): void {
    this.onChange = fn
  }

  registerOnTouched(fn: any): void {
    this.onTouched = fn
  }

  setDisabledState?(isDisabled: boolean): void {
    this.isDisabled = isDisabled
    // 如果你使用的是 OnPush 策略，别忘了下面一行代码
    // this.changeDetectorRef.markForCheck()
  }
}
```

#### 调整 `template`

```ts
  template: `
    <input
      [disabled]="isDisabled"
      [value]="value"
      (input)="onChange($event.target.value)"
      (blur)="onTouched()"
    />
```

### 父组件中使用它

```ts
@Component({
  selector: 'app-root',
  template: `
    简单的双向绑定: <my-input [(value)]="value"></my-input> <br /><br />
    template-form:
    <my-input2 [(ngModel)]="value" [disabled]="true"></my-input2> <br /><br />
    reactive-form: <my-input2 [formControl]="valueControl"></my-input2>
  `,
})
export class AppComponent {
  value = '100'
  valueControl: FormControl

  constructor(fb: FormBuilder) {
    this.valueControl = fb.control(this.value)
    // this.valueControl = fb.control({value:this.value,disabled:true)
    // this.valueControl.enable()
    // this.valueControl.disable()
    this.valueControl.valueChanges.subscribe(value => (this.value = value))
  }
}
```

当你修改第三个输入框的值时，你会发现其他两个输入框的值也发生了改变。

## 表单验证和状态控制

其实当我们实现了 `ControlValueAccessor` 接口的时候，这两个特性就已经实现了。

```html
<!-- app.component.ts template -->
<my-input2 required minlength="4"></my-input2>
<my-input2 [formControl]="validControl"></my-input2>
```

```ts
// app.component.ts constructor
const { required, minLength } = Validators
this.validControl = fb.control(this.value, [required, minLength(4)])
```

```less
// styles.less
.ng-touched.ng-invalid input {
  color: red;
}
```

你会发现它们如期运行。更多的验证器和其他状态的 CSS class 参见[官方文档](https://www.angular.cn/guide/form-validation)

## 组件内的自定义验证

假如我需要让组件只能输入一个 0-9999 数字，输入其他值都是无效的。这个时候我们需要提供 `NG_VALIDATORS` 令牌即可。

让我们来实现这样一个组件: `MyInput3`，省略了与 `MyInput2` 相同的代码

```ts
const defaultMaxValue = 9999
const defaultMinValue = 0
const validateRange = (control: AbstractControl): ValidationErrors => {
  const value = control.value
  const max = defaultMaxValue
  const min = defaultMinValue
  const isValid = value > max || value < min
  return isValid ? null : { range: { value, max, min } }
}

const validatorProvide = {
  provide: NG_VALIDATORS,
  useValue: validateRange,
  multi: true,
}

@Component({
  selector: 'my-input3',
  template: `
    <input
      [disabled]="isDisabled"
      [value]="value"
      (input)="onChange($event.target.value)"
      (blur)="onTouched()"
    />
  `,
  providers: [accessorProvide, validatorProvide],
})
export class MyInput3Component implements ControlValueAccessor {}
```

假如你还需要更好的灵活性，例如你可以让父组件来设置你的最大值和最小值。那你只需要再去实现 `Validator` 即可。

```ts
validate(control: AbstractControl): ValidationErrors // 自定义验证函数
registerOnValidatorChange?(fn: () => void): void // 注册验证状态改变的事件 可选实现
```

```ts
const validatorProvide = {
  provide: NG_VALIDATORS,
  useExisting: forwardRef(() => MyInput3Component),
  multi: true,
}

@Component({ providers: [accessorProvide, validatorProvide] })
export class MyInput4Component implements ControlValueAccessor, Validator {
  @Input() set max(max: number) {
    this._max = max
    this.onValidatorChange()
  }
  get max(): number {
    return this._max
  }
  private _max = 9999

  @Input() set min(min: number) {
    this._min = min
    this.onValidatorChange()
  }
  get min(): number {
    return this._min
  }
  private _min = 0

  private onValidatorChange = () => {}

  validate(control: AbstractControl): ValidationErrors {
    const value = control.value
    const { max, min } = this
    const isValid = value > max || value < min
    return isValid ? null : { range: { value, max, min } }
  }

  registerOnValidatorChange?(fn: () => void): void {
    this.onValidatorChange = fn
  }
}
```

## 更多的 CSS class 控制

可以参考 NG-ZORRO 的实现：[NzFormControlComponent.setControlClassMap](https://github.com/NG-ZORRO/ng-zorro-antd/blob/master/components/form/nz-form-control.component.ts#L120)

## 本文所有完整代码

github: [customized-form-component](https://github.com/danranVm/angular-example/tree/master/customized-form-component)
