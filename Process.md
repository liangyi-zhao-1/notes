## Switch
create a Switch order using API call  or studio wizard  
new order created with pool order id  
trigger 17379 to pool order (place method = FNZClear)  
trigger 151855 to send order to Clear  
log on Clear, and navigate to Deals > managed fund > pre-pooled order to pool order manually  
navigate to Deals > managed fund > pooled order to place order  
on Clear DB, trigger 151857 to send message to oneX to place oneX's order  
on Clear, complete order and authorise completion, trigger 151857 again to send order confirmation info back to OneX to complete order  

- Get instruction, account hierarchy ID, product IDs, holdings
If SmartSwitch -> Process SmartSwitch
If switch version == 3 -> Create V3 switch
***
- OptimisedSwitch
Get optimisedAllocations
foreach optimisedAllocations, Prepare sellInstructions and buyInstructions  
    - ProcessSwitchRequest  
OptimiseSwitchRequest  
CreateSwitchRequestsBatch  
LogOriginalSwitchRequest  
ProcessSwitchRequestBatch => CreateOrders
        - _sellOrderCreator.CreateOrder
***
SubmitInstruction
