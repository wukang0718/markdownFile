```tsx
  namespace FieldSpc {
        interface FieldCfg {
            title: string // 标题
            brief?: string // 描述内容
            disabled?: boolean // 是否禁用区域	默认值	false
            plain?: boolean // 镂空样式	默认值	false
        }

        interface FieldItemCfg {
            title: string // 标题
            content?: string // 描述内容
            addon?: string // 附加文案
            disabled?: boolean // 是否禁用项目	默认值	false
            solid?: boolean // 	是否固定标题宽度，超出会自动换行	默认值	false
            arrow?: boolean // 	动作箭头标识	默认值	false
            placeholder?: string // 提示文字
        }
    }
    
    
    namespace CodeBoxSpc {
        interface CodeBoxCfg {
            maxlength: number // 字符最大输入长度, 若为-1则不限制输入长度 默认值4
            autofocus?: boolean // 是否直通聚焦拉起键盘, 对系统键盘不生效	默认值	false
            mask?: boolean // 是否掩码	默认值	false
            disabled?: boolean // 禁用输入	默认值	false
            justify?: boolean // 自动拉伸布局	默认值	false
            closable?: boolean // 点击输入框及键盘其他区域是否收起键盘	默认值	true
            "ok-text"?: string //  键盘确认键文案	默认值	确认
            disorder?: boolean // 数字键盘是否乱序	默认值	false
            system?: boolean // 是否使用系统默认键盘	默认值	false
            "is-view"?: boolean // 是否内嵌在页面内展示，否则以弹层形式	默认值	false
        }
    }
    
        // md-agree 标签 props
    namespace AgreeSpc {
        interface AgreeCfg {
            disabled?: boolean; // 是否禁用  默认 false
            size?: Size; // 按钮大小  默认 md
            title?: string // 按钮需要需要显示的文字
            position?: IconPosition // 按钮文字需要显示的位置
        }
    }
    
        namespace SliderSpc {
        interface Format {
            (value: any): any
        }

        interface SliderCfg {
            disabled?: boolean // 是否禁用滑块	默认值	false
            min?: number //	可拖动的最小值	 默认值	0
            max?: number //	可拖动的最大值	 默认值	100
            step?: number // 	步长	默认值	1
            range?: boolean // 	是否启动双向拖动	默认值	false
            format?: Format //	显示文本的格式化函数	默认值	(val) => {return val}
        }
    }
    
        namespace SwitchSpc {
        interface SwitchCfg {
            disabled?: boolean // 是否禁用	默认值	false
        }
    }
```



```tsx

    interface AgreeConfigProps extends BaseFormConfigProps {
        options: AgreeSpc.AgreeCfg
    }
    
        interface CodeBoxConfigProps extends BaseFormConfigProps {
        options: CodeBoxSpc.CodeBoxCfg
    }
    
        interface FieldConfigProps extends BaseFormConfigProps {
        options: FieldSpc.FieldCfg
        children: Array<FieldItemConfigProps>
    }
    
    
    interface FieldItemConfigProps extends BaseFormConfigProps {
        options: FieldSpc.FieldItemCfg
    }
    
        interface SliderConfigProps extends BaseFormConfigProps {
        options: SliderSpc.SliderCfg
    }

    interface SwitchConfigProps extends BaseFormConfigProps {
        options: SwitchSpc.SwitchCfg
    }
```





v-validate

provide > node 

solts > errors.length > node > error 

