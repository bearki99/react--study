#  Update

``` javascript
const key = element.key;
if(currentFiber !== null) {
    if(currentFiber.key === key) {
        if(element.$$typeof === REACT_ELEMENT_TYPE) {
            if(currentFiber.type === element.type) {
                // type相同
            }else {
                if(__DEV__) console.warn('还未实现的React类型')
            }
        }else {
            
        }
    }
}

function deleteChild(returnFiber: FiberNode, childToDelete)
```

