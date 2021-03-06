## ISOLATION

Spring Transaction의 격리 수준에 대해서 학습합니다.

### @Transactional

스프링 프레임워크는 이 애너테이션이 붙어 있는 클래스나 메서드에 트랜잭션을 적용합니다. 외부에서 이 클래스의 메서드를 호출할 때 트랜잭션을 시작하고 메서드를 종료할 때 트랜잭션을 커밋합니다. 만약, 예외가 발생하는
경우, 트랜잭션을 롤백합니다.

```
RuntimeException과 그 자식들인 Unchecked Exception만 롤백합니다. 
만약 Checked Exception이 발생해도 롤백하고 싶은 경우, rollbakFor=Exception.class를 옵션으로 명시합니다.
```

### 트랜잭션

트랜잭션은 ACID(Atomicity, Consistency, Isolation, Durability)를 보장해야합니다.

Atomicity(원자성) : 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공 또는 실패해야 한다.

Consistency(일관성) : 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다. 예를들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야 한다.

Isolation(격리성) : 동시에 실행되는 트랜잭션들이 서로에게 영향을 ㅣㅁ치지 않도록 격라한다. 예를 들어 동시에 같은 데이터를 수정하지 모하도록 해야 한다. 격리성은 동시성과 관련된 성능 이슈로 인해 격리
수준을 선택할 수 있다.

Durability(지속성) : 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. 중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.

트랜잭션은 항상 원자성, 일관성, 지속성을 보장합니다. 하지만 격리성의 경우 트랜잭션 간에 격리성을 완벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 합니다. 그러나, 트랜잭션을 매순간 차례대로 수행하면, 동시성
처리 성능이 저하되는 문제가 있습니다.

이러한 문제로 인해, ANSI(미국 국립 표준 협회) 표준은 트랜잭션의 격리 수준을 4단계로 나누어 정의합니다.

- READ UNCOMMITED(커밋되지 않은 읽기)
- READ COMMITED(커밋된 읽기)
- REPEATABLE READ(반복 가능한 읽기)
- SERIALIZABLE(직렬화 가능)

각 Isolation level이 높아질수록, 더 강력한 Lock을 걸 수 있습니다.

트랜잭션 격리는 해당 트랜잭션이 커밋되거나, 롤백될 때 해제됩니다.

|격리 수준|DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
|------|-----------|-------------------|------------|
|READ UNCOMMITTED|O|O|O|
|READ COMMITTED| |O|O|
|REPEATABLE READ| | |O|
|SERIALIZABLE| | | |

격리 수준에 따른 문제점은 다음과 같습니다.

- DIRTY READ
- NON_REPEATABLE READ
- PHANTOM READ

격리 수준이 낮을수록 더 많은 문제가 발생하게 됩니다.

- READ UNCOMMITTED : 커밋되지 않은 데이터를 읽을 수 있습니다. 예를 들어, 트랜잭션이 2개가 동작중일 때, 1번 트랜잭션이 A를 B로 수정하는 중에 2번 트랜잭션이 커밋되지않은 데이터 B를 읽을 수
  있습니다. 이와 같이 현재 커밋되지 않은 데이터를 읽는 것을 DIRTY READ라고 합니다. 만약, 2번 트랜잭션이 B를 READ한 상태에서, 예외 발생으로 인해 1번 트랜잭션이 ROLLBACK을 수행하게되면,
  데이터의 정합성에 심각한 문제가 발생할 수 있습니다.


- READ COMMITTED : 커밋된 데이터만을 읽을 수 있습니다. 즉, DIRTY READ가 발생하지 않습니다. 하지만 NON-REPEATABLE READ는 발생할 수 있습니다. 예를 들어, 1번 트랜잭션이
  회원 A를 조회 중일때 2번 트랜잭션이 회원 A를 수정하고 커밋하면, 1번 트랜잭션이 다시 회원 A를 조회했을 때 수정된 데이터가 조회됩니다. 이처럼 반복해서 같은 데이터를 읽을 수 없는 상태를
  NON-REPEATABLE READ라고 합니다.


- REPEATABLE READ : 한번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회됩니다. 하지만 PHANTOM READ가 발생할 수 있습니다. 예를 들어, 1번 트랜잭션이 회원 테이블로부터 1년미만의
  가입자를 조회하였을 때, 2번 트랜잭션이 새로운 회원을 커밋할 경우, 1번 트랜잭션이 재조회하였을 때, 1개의 ROW가 추가되어 조회됩니다. 이처럼 반복 조회 시, 결과 집합이 달라지는 것을 PHANTOM
  READ라고 합니다.


- SERIALIZABLE : 가장 엄격한 트랜잭션 격리 수준입니다. 발생할 수 있는 문제(DIRTY READ, NON-REPEATABLE READ, PHANTOM READ)가 존재하지 않지만, 격리 수준이 높은
  만큼 동시성 처리 성능이 급격히 떨어집니다.

### 낙관적 잠금(Optimistic Lock)과 비관적 잠금(Pessimistic LOCK)

요청이 많은 서버에서 여러 트랜잭션이 동시에 같은 데이터에 업데이트를 발생시킬 경우에 일부 요청이 유실되는 문제가 발생할 수 있습니다.

엔터프라이즈 애플리케이션의 경우, 데이터베이스에 대한 동시 액세스 문제를 적절하게 관리하는 것이 중요합니다.

JPA의 영속성 컨텍스트(1차 캐시)를 적절히 활용하면 데이터베이스 트랜잭션이 READ COMMITTED 격리 수준이어도 애플리케이션 레벨에서 반복 가능한 읽기가 가능합니다.

낙관적 잠금 : 트랜잭션 대부분은 충돌이 발생하지 않는다고 가정하는 방법입니다. 데이터베이스가 제공하는 락 기능을 사용하지 않고, 어플리케이션 레벨에서 JPA가 제공하는 버전 관리 기능을 사용합니다. 낙관적 잠금의
단점은 커밋하기 전까지는 트랜잭션의 충돌을 알 수 없다는 것입니다.

비관적 잠금 : 트랜잭션의 충돌이 발생한다고 가정하고 우선 락을 걸고 보는 방법입니다. 데이터베이스가 제공하는 락 기능을 사용하며, 대표적으로 select for update 구문이 있습니다.

추가로 데이터베이스 트랜잭션 범위를 넘어선 문제도 존재합니다. A와 B 사용자가 동시에 제목이 같은 게시물을 수정하였을 때, A가 먼저 수정완료 버턴을 눌렀고 바로 뒤 B 가 수정버튼을 눌렀다면
결과적으로 A의 수정사항이 사라지고 B의 수정사항이 반영될 것입니다. 이러한 문제를 **두 번의 갱신 분실 문제(second lost updates problem)** 라고 합니다.

두번의 갱신 분실 문제는 데이터베이스 트랜잭션 범위를 벗어나는 문제이기 때문에, 트랜잭션만으로는 이 문제를 해결할 수 없습니다.
이 때는 아래 3가지의 방법 중 하나를 선택하여 해결합니다.

 - 마지막 커밋만 인정하기 : 사용자 A의 내용은 무시하고 마지막에 커밋한 사용자 B의 내용만 인정
 - 최초 커밋만 인정하기 : 사용자 A가 이미 수정을 완료했으므로 사용자 B가 수정을 온료할 때 오류가 발생
 - 충돌하는 갱신 내용 병합하기 : 사용자 A와 사용자 B의 수정사항을 병합

기본적으로는 마지막 커밋만 인정하기가 사용되지만, 상황에 따라 최초 커밋만 인정하기가 더 합리적일 수도 있습니다. JPA의 버전 관리 기느을 사용하면
손쉽게 최초 커밋만 인정하기를 구현할 수 있습니다.

### @Version

Version 애너테이션은 엔티티의 특정필드레벨에 작성하며, 엔티티를 수정할 때마다 버전이 하나씩 증가합니다. 엔티티를 수정할 때 조회 시험의 버전과 수정 시점의 버전이 다르면 예외가 발생합니다.
또다른 트랜잭션이 해당 값에 대한 조회 및 수정행위가 일어나 버전이 증가하였기 때문에 발생하는 예외입니다.

Version 애너테이션의 적용 가능 타입은 다음과 같습니다.
  - Long (long)
  - Integer (int)
  - Short (short)
  - Timestamp

