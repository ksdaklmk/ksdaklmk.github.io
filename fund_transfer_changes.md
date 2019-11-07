# Change requests in fund transfer/payment
*updated 5-Nobember-2019*

## **Repeating transaction**

This change request impacts all types of fund transfer and payment confirm transactions:
1. PromptPay fund transfer
2. Actual account fund transfer
3. QR payment

### **1. Inquiry request transaction**

By receiving a response returned from sending inquiry request, Mirai to check the following:

1. Check if the returned response status is 200 (success) in order to continue with confirm request
2. If it is success response, Mirai to check further if amount in the respose message is 0.00 before continuing to confirm transaction. In case the amount is 0.00, displays an error message saying "ineligible account, please contact the receiving bank".
3. Else check if such response code is in the list of response codes (81, 87, 99, -201) requiring Mirai to retry sending confirm request for 3 times until success response has been received or the number of retrying has reached the limit
4. Else Mirai to log failed status and display an error message

```python
request.send(inquiryRequest)
if response.status_code == 200:
    if response.amount == 0.00:
        print(errorMessage)
        transaction.log(inquiryRequest, failed)
        break
    else:
        transaction.log(inquiryRequest, success)
        request.send(confirmRequest)
else if response.code in ['99', '81', '87', '-201']:
    tryCount = 1
    while tryCount <= 3:
        request.send(inquiryRequest)
        if response.status_code == 200:
            transaction.log(inquiryRequest, success)
            request.send(confirmRequest)
            break
        else:
            tryCount += 1
    else:
        transaction.log(inquiryRequest, failed)
        print(errorMessage)
else:
    transaction.log(inquiryRequest, failed)
    print(errorMessage)
```

### **2. Confirm request transaction**

By receiving a response returned from sending confirm request, Mirai to check the following:

1. Check if the returned response status is 200 (success) in order to complete the transaction
2. Else check if such response code is in the list of response codes (81, 87, 99, -201) requiring Mirai to retry sending confirm request for 3 times until success response has been received or the number of retrying has reached the limit
3. Else Mirai to reverse the transaction, log failed status and display an error message

```python
request.send(confirmRequest)
if response.status_code == 200:
    transaction.log(confirmRequest, success)
else if response.code in ['99', '81', '87', '-201']:
    tryCount = 1
    while tryCount <= 3:
        request.send(confirmRequest)
        if (response.code = '55') or (response.status_code == 200):
            transaction.log(confirmRequest, success)
            break
        else:
            tryCount += 1
    else:
        transaction.reverse(sourceAccount, transactionAmount)
        transaction.log(confirmRequest, failed)
        print(errorMessage)
else:
    transaction.reverse(sourceAccount, transactionAmount)
    transaction.log(confirmRequest, failed)
    print(errorMessage)
```
