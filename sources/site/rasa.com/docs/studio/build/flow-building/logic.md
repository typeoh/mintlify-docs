# Source: https://rasa.com/docs/studio/build/flow-building/logic

On this page

This guide will teach you how to use **Conditions** in Rasa Studio to make logic branches in your assistant’s conversations. By the end, you’ll know how to set conditions, branch flows, and create fallback options.

### What is a Condition?

Conditions are logical statements used to decide how a conversation proceeds. For example, you can use a condition to check if a slot is filled before showing the next response.

## How to create Logic branching[​](https://rasa.com/docs/studio/build/flow-building/logic/#how-to-create-logic-branching 'Direct link to How to create Logic branching')

1. Select "Logic" from the list to branch your flow.
 
 ![image](https://rasa.com/docs/assets/images/money-transfer-add-logic-faba84029e3b52d425fdb253fbf5b2af.png)
 
2. The branching structure will be immediately created. By clicking "Add logic branch" button you can add as many logic branches as you need.
 
 ![image](https://rasa.com/docs/assets/images/logic-add-branches-46230dc4bccd44ab70550d2f9fb4cf3f.png)
 
3. Click on one of the branches to begin describing the condition(s).
 
 ![image](https://rasa.com/docs/assets/images/select-logic-branch-cb2f03e32f6f62e126fe64ec95fe95ff.png)
 
4. Select the [slot](https://rasa.com/docs/studio/build/flow-building/collect/#slots) from the dropdown menu. You can also create new slots here if they have been used in previously created custom actions. In that case, ensure that the custom action uses the exact slot names created here.
 
 ![image](https://rasa.com/docs/assets/images/logic-select-amount-2b3732088b6b23b3fae8fcc64c49cf0c.png)
 
5. Select an operator. Studio supports the following operators:
 
 - `and`: Combines two conditions with logical AND
 - `or`: Combines two conditions with logical OR
 - `>`: Greater than
 - `>=`: Greater than or equal to
 - `<`: Less than
 - `<=`: Less than or equal to
 - `=`: Equal to
 - `!=`: Not equal to
 - `is`: Checks for identity
 - `is not`: Checks for non-identity
 - `contains`: Checks if a value is contained within another value
 - `matches`: Uses regular expressions to match strings
 - `notmaches`: Uses regular expressions to negate strings
 - `is set`: Checks if the slot has been assigned a value
 - `is empty`: Checks if the slot hasn't been assigned a value
 
 ![image](https://rasa.com/docs/assets/images/logic-select-operator-345fdcb26949df47c9f2e1640f3a8232.png)
 
6. Enter the value
 
 ![image](https://rasa.com/docs/assets/images/logic-select-value-f35fd040904840eed108323aed927f52.png)
 
7. If you want to add more conditions into this branch, click "Add condition". The relationships between the condition is "And" meaning all the conditions need to be met for this branch to move forward.
 
 ![image](https://rasa.com/docs/assets/images/logic-add-condition-a2295959953319c9b060b5d7b16d2025.png)
 

## Else[​](https://rasa.com/docs/studio/build/flow-building/logic/#else 'Direct link to Else')

"Else" is the last logic branch and serves as a backup plan in logic. You don’t need to create any conditions for it but you also can’t delete it. It’s there to make sure the assistant always knows what to do even when no condition that you created for this scenarios matches with what happens in the current dialog.

A next step for "Else" is highly dependent on your use case but it could be a human handoff flow or a generic answer to reply to a user’s answer.

![image](https://rasa.com/docs/assets/images/money-transfer-else-1c64ddb56fde5e74e08d998e835136f6.png)

## Delete conditions[​](https://rasa.com/docs/studio/build/flow-building/logic/#delete-conditions 'Direct link to Delete conditions')

To delete conditions inside Logic step:

- Click the delete icon next to the condition you want to delete. You can delete all but one.
- Confirm the deletion.

![image](https://rasa.com/docs/assets/images/logic-delete-condition-633c992f4395ed60a905d8b58c3fafd0.png)

## Delete logic branches[​](https://rasa.com/docs/studio/build/flow-building/logic/#delete-logic-branches 'Direct link to Delete logic branches')

To delete a logic branch, select it and click the Delete button. You can delete all branches but one.

![image](https://rasa.com/docs/assets/images/logic-delete-branch-9c54be53be7a3c519f8aced3f595a78a.png)

**Warning:** All steps subsequent to this condition will also be deleted. Therefore, we advise you to edit the condition or add new steps before the logic step. Only consider deleting a condition as a last resort.

## Delete Logic step[​](https://rasa.com/docs/studio/build/flow-building/logic/#delete-logic-step 'Direct link to Delete Logic step')

You can delete the entire Logic step the same way you delete any other step—by clicking on the Delete icon. However, a warning will appear notifying you that all conditions and subsequent steps will also be deleted, and you will need to confirm your decision to delete everything.

**Note:** Deleting the entire Logic step can have significant repercussions. We advise you to consider updating the existing logic and conditions or adding new steps before the logic step instead of deleting it.

![image](https://rasa.com/docs/assets/images/delete-logic-2ac0e88cab9d6a0488065e279de7c0f0.png)

- [How to create Logic branching](https://rasa.com/docs/studio/build/flow-building/logic/#how-to-create-logic-branching)
- [Else](https://rasa.com/docs/studio/build/flow-building/logic/#else)
- [Delete conditions](https://rasa.com/docs/studio/build/flow-building/logic/#delete-conditions)
- [Delete logic branches](https://rasa.com/docs/studio/build/flow-building/logic/#delete-logic-branches)
- [Delete Logic step](https://rasa.com/docs/studio/build/flow-building/logic/#delete-logic-step)