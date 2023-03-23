#  createElement实现
``` javascript
function createElement(type, props, ...children) {

}
```

#  render函数
``` javascript
function render(element, container) {
  const dom = element.type === 'TEXT_ELEMENT' ? document.createTextNode("") : document.createElement(element.type)

  element.props.children.forEach(child => render(child, dom));

  container.appendChild(dom);
}
```

#  concurrent
