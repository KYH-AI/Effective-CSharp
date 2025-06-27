# [ITEM_1] 지역변수를 선언할 때는 'var'를 사용하는 것이 낫다

> #### 지역변수의 타입을 암시적으로 선언하는 것이 좋은 이유는
> #### C# 언어가 익명 타입을 지원하기 위해서 타입을 암시적으로 선언할 수 있는 손쉬운 방법을 제공하기 때문이다.  
<br>

정확한 반환 타입을 알지 못한 채 올바르지 않은 타임을 ***명시적***으로 지정하게 되면 득보다 실이 많다.  

코드를 읽은 때도 ***var***를 사용하여 암시적으로 변수를 선언한 코드가 더 쉽게 읽힌다.
- ##### Dictionary<int, Queue<string>>과 같이 정확히 기술된 타입 자체보다는
- ##### jobsQueueByRegion과 같이 타입을 유추할 수 있느 변수의 이름이 더 큰 도움이 된다.


<br>


#### 가독성 문제가 되지 않는 예시
 ```csharp
  var foo = new MyType();
  var thing = AcctounFactory.CreateAccount();
 ```
 - 팩토리 패턴 같이 직관적으로 사용하는 경우 쉽게 어떤 타입으로 반환되는지 확인 가능

#### 가독성 문제가 되는 예시
 ```csharp
   var result = someObject.DoSomeWork(...);
 ```
 - 직관적이 않으면 개발자는 한번 더 해당 타입을 생각해야함

> ***var***를 이용하는 특정 메서드는 반환값을 저장할 변수를 선언할 경우 코드를 해석하기에 조금 어려움을 가져다 준다.<br>
타입을 **명시적**으로 작성한 코드를 읽는 경우는 눈으로 타입을 직접 확인하지만 <br>
***var***를 사용한 경우에는 어떤 타입으로 변환되는지는 직접 확인이 불가능

<br>

## 내장 숫자 타입과 var를 함께 사용한 경우
- 내장된 숫자 타입들 간에는 다양한 변환 연산이 자동으로 수행된다.
- float -> double로의 확대 변환은 안전하게 변환됨
- long -> int로의 축소 변환은 정밀도에 손상
> 숫자를 저장할 변수의 타입을 **명시적**으로 선언하면 변화 과정에서 문제되는 경우 컴파일러 선에서 처리가능

```csharp
 var f = GetMagicNumber();  
 var total = 100 * f / 6; // total은 무슨 타입일까?
```
- *total* 의 정확한 타입은 *GetMagicNumber* 메서드에 결정됨
- 즉 *f* 의 타입에 따라 *total* 계산 시에 사용한 상수들은 동일한 타입으로 변환후 계산됨

#### ⚠️ **var**를 사용하는 컴파일러에게 타입 추론을 위임한 경우
 - 컴파일러는 할당문 오른쪽의 내용을 기반으로 타입을 결정함
 - 형변환 기능때문에 정밀도가 달라짐 (값이 손실됨)
 - 숫자 타입과 **var**를 함께 사용하면 ***가독성*** 문제가 아닌 정확한 값에 문제가 될 수 있다.
> 숫자 타입은 **var** 보다는 **명시적**으로 타입을 선언하고 사용하자

## **var**를 사용하지 않고 사용자가 명시적으로 타입을 어설프게 사용하면 성능 문제를 유발한다.
- DB에서 특정 문자열로 시작하는 고객의 이름을 검색하는 예시
  ```csharp
   public IEnumerable<string> FindCustomers(string str)
  {
    IEnumerable<string> q =
       from c in db.Customers
       select c.ContactName;

     var q2 = q.Where(s => s.StartWith(start));
      return q2;
  }
  ```
  - 해당 코드는 매우 심각한 성능 문제를 유발한다.
  - DB 쿼리가 수행되는 경우 LINQ 쿼리를 실제로 IQueryable<string> 타입을 반환한다.
  - 하지만 *q* 쿼리를 개발자가 IEnumerable<string> 타입을 명시적으로 할당함
  - IQueryable<string> 장점을 이용하지 못함
 
  ### IEnumerable<T> & IQueryable<T>의 개념
   - LINQ-to-Object
     - ㅁㄴㅇㄹ
   - LINQ-to-SQL
     - asdf
