# 使用说明

### 新建活动

随机数需求方，想要生成随机数，首先需要新建一轮活动，需要使用`newCampaign`函数：

```javascript
  function newCampaign(
      uint32 _bnum,
      uint96 _deposit,
      uint8 _commitBalkline,
      uint8 _commitDeadline
  )
```

新建活动需要4个参数：

* `_bnum`：随机数生成的目标块数
* `_deposit`：参与者需要提交的押金
* `_commitBalkline`：开始提交到目标块的距离
* `_commitDeadline`：结束提交到目标块的距离

比如当前块高度是1840602，我们需要在1840900这个块时需要一个随机数，希望每个参与者提交押金为20 ether，在目标块前200个块开始提交（即从1840700块起，包含1840700），在目标块前100个块结束提交（即1840800块截止，包含1840800块），在1840800之后（不包含1840800）1840900块之前（不包含1840900）属于Reveal阶段。可以使用如下方式来调用：

`newCampaign(1840700, 20000000000000000000, 200, 100)`，并且需要发送至少 1 ether 作为参与者奖励费用。

### 跟随活动

随机数需求方可以选择不创建一轮活动，而是选择跟随某一轮随机数活动作为自己的随机数，这时可以使用 Follow 函数。

```javascript
function follow(uint256 _campaignID)
```

`Follow` 函数需要一个参数`_campaignID`，可以使用 Mist 钱包，找到某个campaignID的 `Campaign` 具体信息。

跟随活动必须是在提交随机数窗口期或之前进行，否则就会失败。以前面例子为例，跟随活动必须在1840800块之前（包括1840800）。同样跟随活动需要至少 1 ether 作为参与者的奖励费用。

### 提交随机数

参与者可以通过提交随机数来参与随机数的生成。提交随机可以调用函数：

```javascript
function commit(uint256 _campaignID, bytes32 _hs)
```

`commit` 函数有两个参数：

* `_campaignID`： 活动ID
* `_hs`：随机数的 sha3 值。

提交随机数需要发送押金到合约，不能多于或者少于活动押金，必须刚好等于。提交随机数，必须在提交随机数窗口期提交，否则会失败。以前面例子为例，提交随机数窗口期为：1840700到1840800。

### 披露随机数

```javascript
function reveal(uint256 _campaignID, uint256 _s)
```

`reveal`函数有两个参数：

* `_campaignID`：活动ID
* `_s`：随机数

在随机数提交阶段结束之后，进入Reveal阶段，随机数提交者可以披露自己的随机数，合约会验证是否是有效的随机数，如果有效，将计算到最终的随机数结果中。以前面例子为例，随机数披露窗口期为：1840800到1840900。

### 获取随机数

```javascript
function getRandom(uint256 _campaignID)
```

任何人可以在随机数目标块数之后，获取该轮活动的随机数。只有当所有的随机数提交者提交的随机数全部都收集到，才认为本轮随机数生成有效。对于没有在收集阶段提交随机数的参与者，将罚没其提交的押金，并均分给其他参与者。

### 获取奖励和押金

在目标块之后，随机数提交者可以收回其押金和收益。

* 如果随机数生成成功，将平分奖励费用，并返还押金
* 如果随机数生成失败，将平分未披露随机数的参与者的押金，并返还押金
* 如果随机数生成失败，且没有任何人成功披露随机数，所有人参与者可以取回自己的押金

### 退还奖励

```javascript
function refundBounty(uint256 _campaignID)
```

如果本轮随机数生成失败，随机数需求方可以通过`refundBounty`函数，返还其提交的奖励。