## SOLID

[TOC]

## Single Responsibility Principle

todo

## Open Close Principle

### Before

Handler

```java
public Map<String, Object> pay(String type) {
  Payment payment = new Payment();
  if (type == "credit") {
    payment.payWithCreditCard();
  }
  else if (type == "paypal") {
    payment.payWithPaypal();
  }
  else {
    payment.payWithWireTransfer();
  }
}
```

Payment

```java
public class Payment {
  public void payWithCreditCard() {
    // logic for paying with credit card
  }
  
  public void payWithPaypal() {
   	// logic for paying with paypal 
  }
  
  public void payWithWireCard() {
    // logic for paying with wire transfer
  }
}
```

### After

Payable

```java
interface Payable {
  void pay();
}

public class CreditCardPayment implements Payable {
  public void pay() {
    // execute credit card payment logic
  }
}

public class PaypalPayment implements Payable {
  public void pay() {
    // execute paypal payment logic
  }
}

public class WirePayment implements Payable {
  public void pay() {
    // execute wire payment logic
  }
}
```

PaymentFactory

```java
public class PaymentFactory {
  private Map<String, Payment> payments;
  
  public PaymentFactory() {
    payments = new HashMap(3);
    payments.put("credit", new CreditCardPayment());
    payments.put("paypal", new PaypalPayment());
    payments.put("wire", new WirePayment());
  }
  
  public Payment initialPayment(String type) {
    if (payments.containsKey(type)) {
      return payment.get(type);
    }
    throw new RuntimeException("Unsupported payment method");
  }
}
```

Handler logic

```java
public Map<String, Object> pay(String type) {
  PaymentFactory paymentFactory = new PaymentFactory();
  Payment payment = paymentFactory.initialPayment(type);
  payment.pay();
}
```

## Liskov Substitution Principle

todo 🦆 example

## Interface Segregation Principle

todo

## Dependency Inversion Principle

Never depend on anything concrete, only depend on abstractions.

High level modules should not depend on low level modules. They should depend on abstraction.

Able to change an implementation easily without altering the high level code.

- High level module: handler
- Low level module: db

## Don't get trapped by SOLID

- SOLID design principle are principles, not rules.
- Always use common sense when applying SOLID.
- Avoid over-fragmenting your code for the sake of SRP or SOLID.
- Don't try to achieve SOLID, use SOLID to achieve maintainability.

[Becoming a better developer by using the SOLID design principles by Katerina Trajchevska](https://www.youtube.com/watch?v=rtmFCcjEgEw)
