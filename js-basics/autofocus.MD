# 自动聚焦到输入框末尾

```javascript

export const focus = (element: HTMLInputElement | HTMLTextAreaElement) => {
  element.focus()
  if (typeof getSelection === 'function') {
    element.selectionStart = element.selectionEnd = element.value.length
  }
}

```