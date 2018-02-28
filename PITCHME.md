
## Who Am I?

### Wellington Miranda Barbosa

Cientista da Computação  | Entusiasta 
<center>GoBlockchain.io</center>

---

### SmartContract Multi-Signed

#### A brief example how to implement it.

---
<!-- 
<span class='menu-title'>Show me the Code!</span> -->
```javascript
    /**
    * @dev A struct to hold the Authorizer's informations.
    */
    struct Authorizer {
        address _address;
        uint entryDate;
        uint8 status; // 0 - Inactive | 1 - Active
    }

    /**
    * @dev A struct to hold the Transaction's information about
    * the transfering of tokens from an address to another.
    */
    struct Transaction {
        address from;
        address to;
        bytes32 description;
        uint amount;
        uint date;
        uint8 signatureCount;
        mapping (address => uint8) signatures;
    }

    /**
    * @dev Add the transaction's authorizer.
    * @param authorized The authorized address to be added.
    */
    function addAuthorizer(address authorized) 
      public onlyOwner {
        require(_numberOfAuthorizers <= MAX_AUTHORIZERS);
        require(_authorizers[authorized]._address == 0x0 
                  || _authorizers[authorized].status == 0);

        Authorizer memory authorizer;
        authorizer._address = authorized;
        authorizer.entryDate = now;
        authorizer.status = 1;
        _authorizers[authorized] = authorizer;
        _numberOfAuthorizers++;
    }
    
    /**
    * @dev Request withdraw payment.
    * @param amount The amount to withdraw.
    * @param description The description of the withdraw.
    */
    function requestWithdrawPayment(uint amount, 
                                    bytes32 description) 
    public {
        transferTo(msg.sender, amount, description);
    }

    /**
    * @dev Create a pending transaction to send token 
    * from the Contract's balance to an address.
    * @param to The address to receive the payment.
    * @param amount The amount to withdraw.
    * @param description The description of the withdraw.
    */
    function transferTo(address to, uint amount, 
                              bytes32 description) 
      private {
        require(address(this).balance >= amount);
        uint transactionId = _lastTransactionId++;

        Transaction memory transaction;
        transaction.from = msg.sender;
        transaction.to = to;
        transaction.amount = amount;
        transaction.description = description;
        transaction.date = now;
        transaction.signatureCount = 0;

        _transactions[transactionId] = transaction;
        _pendingTransactions.push(transactionId);

        TransactionCreated(
                              transaction.from, 
                              to, amount, transactionId
                              );
    }

    /**
    * @dev Sign a transaction and if the number of minimum signatures is reached
    * the requested amount is sent to payee.
    * @param transactionId The id of transaction to be signed.
    */
    function signTransaction(uint transactionId)
      public validOwner {
        Transaction storage transaction =
                             _transactions[transactionId];

        // Transaction must exist
        require(0x0 != transaction.from);
        // Creator cannot sign the transaction
        require(msg.sender != transaction.from);
        // Cannot sign a transaction more than once
        require(transaction.signatures[msg.sender] == 0);

        transaction.signatures[msg.sender] = 1;
        transaction.signatureCount++;

        TransactionSigned(msg.sender, 
                                    transactionId);

        if (transaction.signatureCount >= MIN_SIGNATURES) {
            require(address(this).balance >= 
                                        transaction.amount);
            assert(transaction.to.send(transaction.amount));
            TransactionCompleted(
                transaction.from, transaction.to, 
                transaction.amount, transactionId
                );
            deletePendingTransaction(transactionId);
        }
    }
```

<span class="code-presenting-annotation fragment current-only" data-code-focus="1-8"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="9-22"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="25-40"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="42-51"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="53-64"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="66-72"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="74-81"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="84-98"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="100-105"></span>
<span class="code-presenting-annotation fragment current-only" data-code-focus="106-116"></span>

---

## Thank you!

### Questions?

<br>

<i class="fa fa-linkedin gp-contact" aria-hidden="true"> https://linkedin.co/in/wellingtonmb88</i>

<i class="fa fa-github gp-contact" aria-hidden="true"> https://github.com/wellingtonmb88</i>

<i class="fa fa-github gp-contact" aria-hidden="true"> https://goo.gl/YN3ie5</i>
