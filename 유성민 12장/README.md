## 새로운 날짜와 시간 API
  자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했다. 게다가 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스 등 모호한 설계로 유용성이 떨어졌다. 다음은 자바 9의 릴리스 날짜인 2017년 9월 21일을 가리키는 Date 인스턴스를 만드는 코드다.
```java
  Date date = new Date(117, 8, 21);
```
다음은 출력 결과이다:
```
  Thu Sep 21 00:00:00 CET 2017
```
결과가 직관적이지 않고 toString을 활용하기가 어렵다. 그리고 JVM의 기본시간대인 중앙 유럽 시간대를 사용했다. 이렇듯 Date 클래스에 문제가 있다는 점에 의문에 여지가 없으므로 다양한 시간대와 대안 캘린더 등 새로운 날짜와 시간 API를 사용하는 방법을 살펴보자.

### LocalDate, LocalTime, Instant, Duration, Period 클래스
#### LocalDate, LocalTime 사용
  LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체이며 어떤 시간대 정보도 포함하지 않는다. 정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들수 있으며 아래 코드에서 보여주는 것처럼 연도, 달, 요일 등을 반환하는 메서드를 제공한다. 팩토리 메서드인 now는 시스템 시계의 정보를 이용해서 현재 날짜 정보를 얻는다.
```java
    LocalDate date = LocalDate.of(2014, 3, 18);
    int year = date.getYear(); // 2014
    Month month = date.getMonth(); // MARCH
    int day = date.getDayOfMonth(); // 18
    DayOfWeek dow = date.getDayOfWeek(); // TUESDAY
    int len = date.lengthOfMonth(); // 31 (3월의 길이)
    boolean leap = date.isLeapYear(); // false (윤년이 아님)

    LocalDate today = LocalDate.now();
```
  년, 월, 일을 반환하는 내장 메서드도 존재한다.
```java
    int year = date.getYear();
    int month = date.getMonthValue();
    int day = date.getDayOfMonth();
```
  마찬가지로 12:00:00 같은 시간은 LocalTime 클래스로 표현할 수 있다.
```java
    LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
    int hour = time.getHour(); // 13
    int minute = time.getMinute(); // 45
    int second = time.getSecond(); // 20
```
  아래와 같이 parse 정적 메서드를 사용할 수도 있다.
```java
  LocalDate date = LocalDate.parse("2015-02-23");
  LocalTime time = LocalTime.parse("12:02:23");
```

#### 날짜와 시간 조합 (LocalDateTime)
  LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다. 이 클래스는 날짜와 시간을 모두 표현할 수 있으며 다음 코드에서 보여주는 것처럼 직접 만드는 방법도 있고 날짜와 시간을 조합하는 방법도 있다.
```java
    LocalDateTime dt1 = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45, 20); // 2014-03-18T13:45
    LocalDateTime dt2 = LocalDateTime.of(date, time);
    LocalDateTime dt3 = date.atTime(13, 45, 20);
    LocalDateTime dt4 = date.atTime(time);
    LocalDateTime dt5 = time.atDate(date);
```

#### Instant 클래스 : 기계의 날짜와 시간
  Instant 클래스는 유닉스 에포크 시간 (Unix Epoch Time), 1970년 1월 1일 0시 0분 0초 UTC 기준으로 특정 지점까지의 시간을 초로 표현한다. 팩토리 메서드인 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다. 오버로드된 ofEpochSecond 메서드 버전에서 두 번쨰 인수를 이용해서 나노초 단위로 시간을 보정할수있다.
```java
Instant instant = Instant.ofEpochSecond(3);
Instant instant = Instant.ofEpochSecond(3, 0);
Instant instant = Instant.ofEpochSecond(2, 1_000_000_000);
Instant instant = Instant.ofEpochSecond(4. -1_000_000_000);
```
  위 네 코드는 같은 Instant를 반환한다.

#### Duration과 Period 정의
  Duration 클래스의 정적 팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다. 두 개의 LocalTime, LocalDateTime, 또는 Instant로 Duration을 마들 수 있다.
```java
    Duration d1 = Duration.between(dateTime1, dateTime2);
    Duration d2 = Duration.between(instant1, instant2);
```
  Duration 클래스는 초와 나노초로 시간 단위를 표현하므로 between 메서드에 LocalDate을 전달할 수 없다. Period 클래스에 between을 사용하면 두 LocalDate의 차이를 확인할 수 있다.
```java
  Period tenDays = Period.between(LocalDate.of(2017, 99, 11), LocalDate.of(2017, 9, 21));
```

### 날짜 조정, 파싱, 포매팅
  어떤 Temporal 객체가 지정된 필드를 지원하지 않으면 UnsupportedTemporalTypeException이 발생한다. 예를 들어 Instant에 ChronoField.MONTH_OF_YEAR를 사용하거나 LocalDate에 ChronoField.NANO_OF_SECOND를 사용하면 예외가 발생한다.
  절대적인 방식으로 LocalDate의 속성 바꾸기
```java
  LocalDate date1 = LocalDate.of(2017, 9, 21);
  LocalDate date2 = date1.withYear(2011);
  LocalDate date3 = date2.withDayOfMonth(25);
```
  상대적인 방식으로 LocalDate의 속성 바꾸기
```java
  LocalDate date1 = LocalDate.of(2017, 9, 21);
  LocalDate date2 = date1.plusWeeks(1);
```

#### TemporalAdjusters 사용하기
  지금까지 살펴본 날짜 조정 기능들은 매우 간단하다, 하지만 때로는 다음 주 일요일, 돌아오늘 평일 같이 좀 더 복잡한 날짜 조정 기능이 필요할것이다. 이때는 TemporalAdjuster를 사용해서 조정할 수 있다.

Method
  firstDayOfNextYear()	  다음 해의 첫 날
  firstDayOfNextMonth()	  다음 달의 첫 날
  firstDayOfYear()        올 해의 첫 날
  firstDayOfMonth()	      이번 달의 첫 날
  lastDayOfYear()	        올 해의 마지막 날
  lastDayOfMonth()	      이번 달의 마지막 날
  firstInMonth(DayOfWeek dayOfWeek)	    이번 달의 첫 번째 요일
  lastInMonth(DayOfWeek dayOfWeek)	    이번 달의 마지막 요일
  previous(DayOfWeek dayOfWeek)	        지난 요일(당일 미포함)
  previousOrSame(DayOfWeek dayOfWeek)	  지난 요일(당일 포함)
  next(DayOfWeek dayOfWeek)	            다음 요일(당일 미포함)
  nextOrSame(DayOfWeek dayOfWeek)	      다음 요일(당일 포함)
  dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek)	  이번 달의 n번째 요일

  TemporalAdjuster 인터페이스는 하나의 메서드만 정의하는 함수형 인터페이스이다. 그렇기 때문에 비교적 쉽게 커스텀 TemporalAdjuster를 구현할 수 있다. TemporalAdjuster 인터페이스 구현은 Temporal 객체를 어떻게 다른 Temporal 객체로 변환할지 정의한다.
  아래는 다음 WorkingDay (평일)을 구하는 커스텀 TemporalAdjuster이다:
```java
  private static class NextWorkingDay implements TemporalAdjuster {

    @Override
    public Temporal adjustInto(Temporal temporal) {
      DayOfWeek dow = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));
      int dayToAdd = 1;
      if (dow == DayOfWeek.FRIDAY) {
        dayToAdd = 3;
      }
      if (dow == DayOfWeek.SATURDAY) {
        dayToAdd = 2;
      }
      return temporal.plus(dayToAdd, ChronoUnit.DAYS);
    }
  }
```

#### DateTimeFormatter
  기존의 DateFormat 클래스와 달리 모든 DateTimeFormatter는 스레드에서 안전하게 사용할 수 있는 클래스다. 그리고 아래와 같이 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드도 제공한다.
```java
    LocalDate date = LocalDate.of(2014, 3, 18);
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    DateTimeFormatter italianFormatter = DateTimeFormatter.ofPattern("d. MMMM yyyy", Locale.ITALIAN);

    System.out.println(date.format(DateTimeFormatter.ISO_LOCAL_DATE));
    System.out.println(date.format(formatter));
    System.out.println(date.format(italianFormatter));
```
  DateTimeFormatterBuilder 클래스로 좀 더 세부적으로 포메터를 제어하는것도 가능하다.
```java
    DateTimeFormatter complexFormatter = new DateTimeFormatterBuilder()
        .appendText(ChronoField.DAY_OF_MONTH)
        .appendLiteral(". ")
        .appendText(ChronoField.MONTH_OF_YEAR)
        .appendLiteral(" ")
        .appendText(ChronoField.YEAR)
        .parseCaseInsensitive()
        .toFormatter(Locale.ITALIAN);
```
