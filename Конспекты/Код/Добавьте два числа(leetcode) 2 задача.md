#leetcode #Код
![[{0981B6B6-5131-48D7-9FC6-A9E59449149B}.png]]

```python
#Зимин Я.В.

class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy = ListNode(0)
        curr = dummy
        carry = 0 # Переменная для хранения переноса если сумма чисел больше 9
        
        while l1 or l2 or carry:
	        # Получаем значения текущих узлов l1 и l2
            v1 = l1.val if l1 else 0
            v2 = l2.val if l2 else 0
            # Суммируем текущие значения и перенос
            total = v1 + v2 + carry
            carry = total // 10
            
	         # Создаем новый узел с результатом
            curr.next = ListNode(total % 10)
            curr = curr.next
            # Двигаем указатели l1 и l2 к следующим узлам
            if l1:
                l1 = l1.next
            if l2:
                l2 = l2.next
        # Возвращаем результат
        return dummy.next
```