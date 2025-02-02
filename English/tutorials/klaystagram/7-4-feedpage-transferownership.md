## 7-4. TransferOwnership component

![transfer ownership](./images/klaystagram-transferownership.png)

1. `TransferOwnership` component's role
2. Component code
    2-1. Rendering `transferOwnership` button
    2-2. `TransferOwnership` component
3. Interact with contract: `transferOwnership` method  
4. Update data to store: `updateOwnerAddress` action  

### 1) `TransferOwnership` component's role
The owner of photo can transfer photo's ownership to another user. By sending `transferOwnership` transaction, new owner's address will be saved in ownership history, which keep tracks of past owner addresses.

### 2) Component code

#### 2-1) Rendering `TransferOwnership` button
We are going to render `TransferOwnership` button on the `FeedPhoto` component only when photo's owner address matches with logged-in user's address (which means you are the owner).

```js
// src/components/Feed.js

<div className="FeedPhoto">
  // ...
  {
    userAddress === currentOwner && (
      <TransferOwnershipButton
        className="FeedPhoto__transferOwnership"
        id={id}
        issueDate={issueDate}
        currentOwner={currentOwner}
      />
    )
  }
  // ...
</div>
```

#### 2-2) `TransferOwnership` component

```js
// src/components/TransferOwnership.js

import React, { Component } from 'react'
import { connect } from 'react-redux'
import * as photoActions from 'redux/actions/photos'
import ui from 'utils/ui'
import { isValidAddress } from 'utils/crypto'
import Input from 'components/Input'
import Button from 'components/Button'

import './TransferOwnership.scss'

class TransferOwnership extends Component {
  state = {
    to: null,
    warningMessage: '',
  }

  handleInputChange = (e) => {
    this.setState({
      [e.target.name]: e.target.value,
    })
  }

  handleSubmit = (e) => {
    e.preventDefault()
    const { id, transferOwnership } = this.props
    const { to } = this.state

    if (!isValidAddress(to)) {
      return this.setState({
        warningMessage: '* Invalid wallet address',
      })
    }
    transferOwnership(id, to)
    ui.hideModal()
  }

  render() {
    const { id, issueDate, currentOwner } = this.props
    return (
      <div className="TransferOwnership">
        <h3 className="TransferOwnership__copyright">Copyright. {id}</h3>
        <p className="TransferOwnership__issueDate">Issue Date  {issueDate}</p>
        <form className="TransferOwnership__form" onSubmit={this.handleSubmit}>
          <Input
            className="TransferOwnership__from"
            name="from"
            label="Current Owner"
            value={currentOwner}
            readOnly
          />
          <Input
            className="TransferOwnership__to"
            name="to"
            label="New Owner"
            onChange={this.handleInputChange}
            placeholder="Transfer Ownership to..."
            err={this.state.warningMessage}
            required
          />
          <Button
            type="submit"
            title="Transfer Ownership"
          />
        </form>
      </div>
    )
  }
}

const mapDispatchToProps = (dispatch) => ({
  transferOwnership: (id, to) => dispatch(photoActions.transferOwnership(id, to)),
})

export default connect(null, mapDispatchToProps)(TransferOwnership)
```

### 3) Interact with contract: `transferOwnership` method  
We already made `transferOwnership` function in Klaystagram contract at chapter [4. Write Smart Contract](./4-write-smart-contract.md). Let's call it from application.

1. Invoke the contract method: `transferOwnership`  
    * `id:` Photo's tokenId
    * `to:` Address to tranfer photo's ownership
2. Set transaction options 
    * `from`: An account that sends this transaction and pays the transaction fee.  
    * `gas`: The maximum amount of gas that the `from` account is willing to pay for this transaction.
3. After sending the transaction, show progress along the transaction lifecycle using `Toast` component.
4. If the transaction successfully gets into a block, call `updateOwnerAddress` function to update new owner's address into the feed page.

```js
// src/redux/actions/photo.js

export const transferOwnership = (tokenId, to) => (dispatch) => {
  /** 
   * 1. Invoke the contract method: `transferOwnership`
   * id: photo's token id
   * to: new owner's address
  */
  KlaystagramContract.methods.transferOwnership(tokenId, to).send({
    // 2. Set transaction options
    from: getWallet().address,
    gas: '20000000',
  })
    .once('transactionHash', (txHash) => {
      // 3. After sending the transaction,
      // show progress along the transaction lifecycle using `Toast` component.
      ui.showToast({
        status: 'pending',
        message: `Sending a transaction... (transferOwnership)`,
        txHash,
      })
    })
    .once('receipt', (receipt) => {
      ui.showToast({
        status: receipt.status ? 'success' : 'fail',
        message: `Received receipt! It means your transaction is
          in klaytn block (#${receipt.blockNumber}) (transferOwnership)`,
        link: receipt.transactionHash,
      })

      // 4. If the transaction successfully gets into a block,
      // call `updateOwnerAddress` function to update new owner's address into the feed page.
      dispatch(updateOwnerAddress(tokenId, to))
    })
    .once('error', (error) => {
      ui.showToast({
        status: 'error',
        message: error.toString(),
      })
    })
}
```

### 4) Update information in redux store: `updateOwnerAddress` action

After transfering ownership, FeedPhoto needs to be rerendered with new owner's address.  
To update new owner's address, let's call `feed` data from store and find the photo that has the tokenId from the receipt. Then push new owner's address to photo's `OWNER_HISTORY` and setFeed.

```js
const updateOwnerAddress = (tokenId, to) => (dispatch, getState) => {
  const { photos: { feed } } = getState()
  const newFeed = feed.map((photo) => {
    if (photo[ID] !== tokenId) return photo
    photo[OWNER_HISTORY].push(to)
    return photo
  })
  dispatch(setFeed(newFeed))
}
```
