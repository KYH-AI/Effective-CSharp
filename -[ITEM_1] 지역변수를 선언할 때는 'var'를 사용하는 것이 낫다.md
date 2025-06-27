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
   - #### LINQ-to-Object
     - LINQ 쿼리를 메모리에서 로컬로 실행
     - 쿼리는 DB에 모든 데이터를 가져와 메모리에서 필터 및 정렬 수행
   - #### LINQ-to-SQL
     - LINQ 쿼리를 DB에서 모든걸 수행하도록 SQL 쿼리로 변환
     - IQueryable<T>는 IEnumrable<T>를 상속받음

  <br>
    
  | 구분 | LINQ to Objects | LINQ to SQL |
  | --- | --- | --- |
  | 작동 위치 | 로컬 메모리 (RAM) | 원격 DB (SQL Server 등) |
  | 데이터 소스 |	배열, List<T>, Dictionary 등 |	데이터베이스 (EF, SQL Server 등)
  | 처리 방식 | .NET이 메모리 내에서 직접 연산 | 쿼리를 SQL로 번역해서 DB에 위임 |
  | 성능 특징 | 작은 데이터엔 빠름	| 대용량에 최적화 (서버가 처리) |
  | 대표 인터페이스 | IEnumerable<T> | IQueryable<T> |
  
  ![image](https://github.com/user-attachments/assets/6e5701d9-5f09-4299-ba02-e8aef02c6833)

   <br>

   #### var을 사용하지 않았기 때문에...
   ```csharp
     IEnumerable<string> q =
       from c in db.Customers
       select c.ContactName;

      var q2 = q.Where(s => s.StartWith(start));
   ```
   - **var**이 아닌 명시적 타입으로 선언하여 모든 DB에서 필터링되지 않은 연락처와 이름을 로컬에 읽어온다.
   - **Where** 두 번쨰 쿼리문 또한 로컬 메모리에서 필터링이 진행된다.

   #### var을 사용했기 떄문에...

    ```csharp
        var q =
       from c in db.Customers
       select c.ContactName;

      var q2 = q.Where(s => s.StartWith(start));
    ```
     - **var**을 이용해 런타임에서 IQueryable<string>으로 자동으로 변환
     - ___ **첫 번째**___  쿼리가 실행되면서 DB에 SQL이 전달되지 않고 ___ **두 번째**___ 쿼리까지 조합하여 전달된다. ___(지연실행(Lazy execution))___
     - 조합된 쿼리가 DB에 SQL로 전달되어 필터링된 데이터를 얻는다. 
