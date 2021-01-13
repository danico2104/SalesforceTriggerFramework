# SalesforceTriggerFramework

There are a lot of opinions around this topic but I would like to tell you my experience through this repository as a good practice


## Overview
Triggers are a widely used functionality, but as the project progresses and becomes more established, its disorderly use causes inadvertent situations or unwanted side effects, with frustration on the part of the developer and the user.

But for this example is really important to have the KISS principle always in mind.

## Usage

To create a trigger handler, you simply need to create a class that inherits from **TriggerHandler.cls**. Here is an example for creating an Account trigger handler.

```java
public class AccountTriggerHandler extends TriggerHandler {}
```

Always make sure that your logic override the standard methods that we have in salesforce. you know, Before and After methods

for example


```java
public class AccountTriggerHandler extends TriggerHandler {
  
  public override void beforeUpdate() {
    for(Account accObj : (List<Account>) Trigger.new) {
      // do something cool and make sure always follow good practices
    }
  }
}
```


**Note:** When referencing the Trigger statics within a class, SObjects are returned versus SObject subclasses like Account, Opportunity etc. This means that you must cast when you reference them in your trigger handler. You could do this in your constructor if you wanted.

```java
public class AccountTriggerHandler extends TriggerHandler {

  private Map<Id, Account> newAccMap;

  public AccountTriggerHandler() {
    this.newAccMap = (Map<Id, Account>) Trigger.newMap;
  }
  
  public override void afterUpdate() {}

}
```

To use the trigger handler, you only need to construct an instance of your trigger handler within the trigger handler itself and call the `run()` method. Here is an example of the Account trigger.

```java
trigger AccountTrigger on Account (after insert, after update, after delete, before insert, before update, before delete, after undelete) {
  new AccountTriggerHandler().run();
}
```


##Features

### Max Loop Count
To prevent recursion, you can set a max loop count for Trigger Handler. If this max is exceeded, and exception will be thrown. A great use case is when you want to ensure that your trigger runs once and only once within a single execution. Example:


```java
public class AccountTriggerHandler extends TriggerHandler {

  public AccountTriggerHandler() {
    this.setMaxLoopCount(1);
  }
  
  public override void afterUpdate() {
    List<Account> opps = [SELECT Id FROM Account WHERE Id IN :Trigger.newMap.keySet()];
    update opps; // this will throw after this update
  }

}
```

### Trigger Bypass

There are some times that you don't need to run this trigger, so this bypass make sure not execute logic if not needed.

```java
public class OpportunityTriggerHandler extends TriggerHandler {
  
  public override void afterUpdate() {
    List<Opportunity> opps = [SELECT Id, AccountId FROM Opportunity WHERE Id IN :Trigger.newMap.keySet()];
    
    Account acc = [SELECT Id, Name FROM Account WHERE Id = :opps.get(0).AccountId];

    TriggerHandler.bypass('AccountTriggerHandler');

    acc.Name = 'No Trigger';
    update acc; // won't invoke the AccountTriggerHandler

    TriggerHandler.clearBypass('AccountTriggerHandler');

    acc.Name = 'With Trigger';
    update acc; // will invoke the AccountTriggerHandler

  }

}
```

If you need to check if a handler is bypassed, use the `isBypassed` method:

```java
if (TriggerHandler.isBypassed('AccountTriggerHandler')) {
  // ... do something if the Account trigger handler is bypassed!
}
```

If you want to clear all bypasses for the transaction, simple use the `clearAllBypasses` method, as in:

```java
// ... done with bypasses!

TriggerHandler.clearAllBypasses();

// ... now handlers won't be ignored!
```

## Overridable Methods

Here are all of the methods that you can override. All of the context possibilities are supported.

* `beforeInsert()`
* `beforeUpdate()`
* `beforeDelete()`
* `afterInsert()`
* `afterUpdate()`
* `afterDelete()`
* `afterUndelete()`
