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


Collab接收到其他端的更新时

- 先调用this.reconcileElements(decryptedData.payload.elements)协调元素
- this.handleRemoteSceneUpdate()