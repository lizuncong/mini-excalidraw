## 整体设计
几大组件的作用

## socket事件设计

## 消息同步


## 进入会议流程



## 第一个进入房间
第一个进入房间的用户包括以下两种场景：
1.创建会议。将本地元素数据保存到firebase
2.已经创建了会议，然后刷新浏览器。这种场景下从firebase获取元素数据


## Collab组件
```js
broadcastElements = (elements: readonly ExcalidrawElement[]) => {
    console.log('Collab组件broadcastElements', getSceneVersion(elements))
    if (
        getSceneVersion(elements) >
        this.getLastBroadcastedOrReceivedSceneVersion()
    ) {
        console.log('Collab组件调用broadcastScene开始广播')
        this.portal.broadcastScene(WS_SCENE_EVENT_TYPES.UPDATE, elements, false);
        this.lastBroadcastedOrReceivedSceneVersion = getSceneVersion(elements);
        this.queueBroadcastAllElements();
    }
};
```
由于只要App组件状态发生变化都会触发componentDidUpdate，但只有元素发生变化才需要广播元素数据，因此才有了if判断。

`this.portal.broadcastScene`里面会判断元素是否符合广播的条件，需要同时满足以下两个条件：

- element.version > this.broadcastedElementVersions.get(element.id)，通过版本号判断元素是否被修改了，只将被修改的元素广播出去
- isSyncableElement(element)是否是可同步的元素，比如宽度高度太小的元素不广播

>this.portal.broadcastScene在广播元素时，会给元素添加一个PRECEDING_ELEMENT_KEY属性，

Collab接收到其他端的更新时

- 先调用this.reconcileElements(decryptedData.payload.elements)协调元素
- this.handleRemoteSceneUpdate()


## reconciliation协调算法
- 1.首先将本地元素和他们对应的索引映射成map，比如：

```js
// 假设localElements如下：
const localElements = [A, B, C, D]
// 那么映射后的map如下：
const localElementsData = {
    [A.id]: [A, 0], // 数组第一项存的是元素自身，第二项存的是元素的索引
    [B.id]: [B, 1],
    [C.id]: [C, 2],
    [D.id]: [D, 3],
}
```

- 2.遍历每一个remoteElement元素
    + 调用shouldDiscardRemoteElement(localAppState, localElement, remoteElement)判断是否要丢弃合并远程更新。以下三种情况需要丢弃远程更新
        + 本地元素正在编辑，包括正在修改元素、缩放元素、拖拽元素
        + 本地元素的verson大于remote.version
        + 本地元素和远程元素version相同，同时本地元素的versionNonce小于远程元素的versionNonce
      如果不能丢弃远程元素，则说明需要将远程元素更新到本地，则需要将本地元素删除