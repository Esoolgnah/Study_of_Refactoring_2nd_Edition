# :star: 1-4 `statement()` 함수 쪼개기

<br>

[예시 코드](https://github.com/Esoolgnah/Summary_of_Refactoring_2nd_Edition/blob/main/01_리팩터링_첫번째_예시/01_01_자_시작해보자.md)의 switch 문을 보면 한 번의 공연에 대한 요금을 계산하고 있습니다. 여기서는 코드 조각을 별도 함수로 추출하는 방식으로 앞서 파악한 정보를 코드에 반영할 것입니다. 추출한 함수에는 그 코드가 하는 일을 설명하는 이름을 지어줍니다. `amountFor(aPerformance)` 정도면 적당해 보입니다. 이 절차를 따로 기록해두고, 나중에 참조하기 쉽도록 `함수 추출하기`라는 이름을 붙였습니다.

<br>

먼저 별도 함수로 빼냈을 때 유효범위를 벗어나는 변수, 즉 새 함수에서는 곧바로 사용할 수 없는 변수(`perf`, `play`, `thisAmount`)가 있는지 확인합니다. `perf`와 `play`는 값을 변경하지 않기 때문에 매개변수로 전달하면 됩니다. 함수 안에서 값이 바뀌는 변수(`thisAmount`)는 조심해서 다뤄야합니다. 이번 예에서는 이런 변수가 하나뿐이므로 이 값을 변환하도록 작성했습니다. 또한 이 변수를 초기화하는 코드도 추출한 함수에 넣었습니다. 결과는 다음과 같습니다.

```js
// statement() 함수...

function amountFor(perf, play) {
  // 값이 바뀌지 않는 변수는 매개변수로 전달
  let thisAmount = 0; // 변수를 초기화하는 코드

  switch (play.type) {
    case 'tragedy': // 비극
      thisAmount = 40000;
      if (perf.audience > 30) {
        thisAmount += 1000 * (perf.audience - 30);
      }
      break;
    case 'comedy': // 희극
      thisAmount = 30000;
      if (perf.audience > 20) {
        thisAmount += 10000 + 500 * (perf.audience - 20);
      }
      thisAmount += 300 * perf.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${play.type}`);
  }
  return thisAmount; // 함수 안에서 값이 바뀌는 변수 반환
}
```

<br>

이제 `statement()` 에서는 `thisAmount` 값을 채울 때 방금 추출한 `amountFor()` 함수를 호출합니다.

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;

  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = amountFor(perf, play); // 추출한 함수를 이용

    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (play.type === 'comedy') volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${play.name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

아무리 간단한 수정이라도 리팩터링 후에는 항상 테스트하는 습관을 들이는 것이 바람직합니다. `조금씩 변경하고 매번 테스트하는 것이 리팩터링 절차의 핵심입니다.` 한번에 너무 많이 수정하려다 실수를 저지르면 디버깅하기 어려워서 결과적으로 작업 시간이 늘어납니다.

<br>

<br>

> 리팩터링은 프로그램 수정을 작은 단계로 나눠 진행합니다. 그래서 중간에 실수하더라도 버그를 쉽게 찾을 수 있습니다.

<br>

<br>

지금 예는 자바스크립트 코드이므로 `amountFor()`를 `statement()`의 중첩함수로 만들 수 있었습니다. 이렇게 바깥 함수에서 쓰던 변수를 새로 추출한 함수에 매개변수로 전달할 필요가 없어 편합니다. 일반적으로는 중첩 함수로 만들면 할 일 하나가 줄어드는 셈입니다.

<br>

하나의 리팩터링을 문제없이 끝낼 때마다 커밋합니다. 그래야 중간에 문제가 생기더라도 이전의 정상 상태로 쉽게 돌아갈 수 있습니다. 어느 정도 의미 있는 단위로 뭉쳐지면 공유저장소로 푸시합니다.

<br>

변수의 이름을 더 명확하게 바꿔봅시다. 가령 `thisAmount`를 `result`로 변경할 수 있습니다. 필자는 함수의 반환 값에는 항상 `result`라는 이름을 씁니다. 그러면 그 변수의 역할을 쉽게 알 수 있기 때문입니다. 인수인 `perf`는 `aPerformance`라고 바꿔주는 등 명확하게 바꿔줍니다. 필자는 매개변수 이름에 접두어로 `타입이름`을 적는데, 매개변수의 역할이 뚜렷하지 않을 때는 `부정 관사(a/an)`을 붙입니다.

<br>

```js
// statement() 함수...

function amountFor(aPerformance, play) {
  // perf => aPerformance : 명확한 이름으로 변경
  let result = 0; // thisAmount => result : 명확한 이름으로 변경

  switch (play.type) {
    case 'tragedy': // 비극
      result = 40000; // thisAmount
      if (aPerformance.audience > 30) {
        // perf
        result += 1000 * (aPerformance.audience - 30); // thisAmount, perf
      }
      break;
    case 'comedy': // 희극
      result = 30000; // thisAmount
      if (aPerformance.audience > 20) {
        // perf
        result += 10000 + 500 * (aPerformance.audience - 20); // thisAmount, perf
      }
      result += 300 * aPerformance.audience; // thisAmount, perf
      break;
    default:
      throw new Error(`알 수 없는 장르: ${play.type}`);
  }
  return result; // thisAmount
}
```

<br>

<br>

> 컴퓨터가 이해하는 코드는 바보도 작성할 수 있습니다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자입니다.

<br>

<br>

좋은 코드라면 하는 일이 명확히 드러나야 하며, 이때 변수 이름은 커다란 역할을 합니다. 그러니 명확성을 높이기 위한 이름 바꾸기에는 조금도 망설이지 맙시다. `찾아 바꾸기` 기능을 제공하는 도구를 사용하면 어렵지 않습니다. 다음으로 `play` 매개변수의 이름을 바꿀 차례입니다. 그런데 이 변수는 좀 다르게 처리해야 합니다.

<br>

### `play` 변수 제거하기

긴 함수를 쪼갤 때는 필요 없는 임시 변수를 최대한 제거합니다. 임시 변수들 때문에 로컬 범위에 존재하는 이름이 늘어나서 추출 작업이 복잡해지기 때문입니다. 이를 해결해주는 리팩터링으로는 `임시 변수를 질의 함수로 바꾸기`가 있습니다. 먼저 대입문(=)의 우변을 함수로 추출합니다.

<br>

```js
// statement() 함수...
function playFor(aPerformance) {
  return plays[aPerformance.playID];
}

// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    const play = playFor(perf); // 우변을 함수로 추출
    let thisAmount = amountFor(perf, play);

    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (play.type === 'comedy') volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${play.name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

`컴파일-테스트-커밋`한 다음 `변수 인라인하기`를 적용합니다.

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    // const play = playFor(perf); 인라인된 변수는 제거합니다.
    let thisAmount = amountFor(perf, playFor(perf));

    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      // <= 변수 인라인, play: playFor(perf)
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`; // 변수 인라인, play: playFor(perf)
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

다시 `컴파일-테스트-커밋`합니다. 변수를 인라인한 덕분에 `amountFor()`에 함수 선언 바꾸기를 적용해서 `play` 매개변수를 제거할 수 있게 되었습니다. 이 작업은 두 단계로 진행합니다. 먼저 새로 만든 `playFor()`를 사용하도록 `amountFor()`를 수정합니다.

<br>

```js
// statement() 함수...

function amountFor(aPerformance, play) {
  let result = 0;

  switch (
    playFor(aPerformance).type // play를 playFor() 호출로 변경
  ) {
    case 'tragedy': // 비극
      result = 40000; // thisAmount
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case 'comedy': // 희극
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${playFor(aPerformance).type}`); // play를 playFor() 호출로 변경
  }
  return result;
}
```

<br>

`컴파일-테스트-커밋`하고 `play` 매개변수를 삭제합니다.

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    // let thisAmount = amountFor(perf,playFor(perf));
    let thisAmount = amountFor(perf); // 필요 없어진 매개변수 제거

    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(thisAmount / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

```js
// statement() 함수...

// function amountFor(aPerformance,play) {
function amountFor(aPerformance) {
  // 필요 없어진 매개변수 제거
  let result = 0;

  switch (playFor(aPerformance).type) {
    case 'tragedy': // 비극
      result = 40000; // thisAmount
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case 'comedy': // 희극
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
      throw new Error(`알 수 없는 장르: ${playFor(aPerformance).type}`);
  }
  return result;
}
```

<br>

다시 한번`컴파일-테스트-커밋`합니다. 방금 수행한 리팩터링에서 주목할 점이 몇 가지 있습니다. 이전 코드는 루프를 한 번 돌 때마다 공연을 조회한데 반해 리팩터링한 코드에서는 세 번이나 조회합니다. 뒤에서 자세히 설명하겠지만 이렇게 변경해도 `성능에 큰 영향은 없습니다.` 설사 심각하게 느려지더라도 제대로 리팩터링된 코드베이스는 그렇지 않은 코드보다 `성능을 개선하기 훨씬 수월합니다.`

<br>

지역 변수를 제거해서 얻는 가장 큰 장점은 `추출 작업이 훨씬 쉬워진다는 것입니다.` 유효범위를 신경 써야 할 대상이 줄어들기 때문입니다. `amountFor()`에 전달할 인수를 모두 처리했으니, 이 함수를 호출하는 코드로 돌아가봅시다. 여기서 `amountFor()`는 임시 변수인 `thisAmount`에 값을 설정하는 데 사용되는데, 그 값이 다시 바뀌지는 않습니다. 따라서 `변수 인라인하기`를 적용합니다.

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    let thisAmount = amountFor(perf);

    // 포인트를 적립한다.
    volumeCredits += Math.max(perf.audience - 30, 0);

    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if (playFor(perf).type === 'comedy')
      volumeCredits += Math.floor(perf.audience / 5);

    // 청구 내역을 출력한다.
    // result += `  ${playFor(perf).name}: ${format(thisAmount / 100)} (${perf.audience}석)\n`; <= thisAmount 변수를 인라인: amountFor(perf)
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    // totalAmount += thisAmount;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

### 적립 포인트 계산 코드 추출하기

<br>

앞에서 `play` 변수를 제거한 결과 로컬 유효범위의 변수가 하나 줄어서 적립 포인트 계산 부분을 추출하기가 훨씬 쉬워졌습니다. 처리해야 할 변수가 아직 두 개 더 남아 있습니다. 여기서도 `perf`는 간단히 전달만 하면 됩니다. 하지만 `volumeCredits`는 반복문을 돌 때마다 값을 누적해야 하기 때문에 살짝 더 까다롭습니다. 여기서 최선의 방법은 추출한 함수에서 `volumeCredits`의 복제본을 초기화한 뒤 계산 결과를 반환토록 하는 것입니다.

<br>

<br>

```js
// statement() 함수...
function volumeCreditsFor(perf) {
  // 새로 추출한 함수
  let volumeCredits = 0;
  volumeCredits += Math.max(perf.audience - 30, 0);
  if ('comedy' === playFor(perf).type)
    volumeCredits += Math.floor(perf.audience / 5);
  return volumeCredits;
}
```

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf); // 추출한 함수를 이용해 값을 누적

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

`컴파일-테스트-커밋`한 다음, 새로 추출한 함수에서 쓰이는 변수들 이름을 적절히 바꿉니다.

<br>

```js
// statement() 함수...
function volumeCreditsFor(aPerformance) {
  // perf => aPerformance
  let result = 0; // volumeCredits => result
  result += Math.max(aPerformance.audience - 30, 0); // volumeCredits, perf
  if ('comedy' === playFor(aPerformance).type)
    // perf
    result += Math.floor(aPerformance.audience / 5); // volumeCredits, perf
  return result; // volumeCredits
}
```

<br>

이상의 작업을 하나의 단계처럼 표현했지만 앞과 마찬가지로 변수 이름을 하나씩 바꿀 때마다 `컴파일-테스트-커밋`했습니다.

<br>

### `format` 변수 제거하기

<br>

다시 최상위 코드인 `statement()`를 살펴봅시다.

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  const format = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount / 100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

임시 변수는 나중에 문제를 일으킬 수 있습니다. 또한 자신이 속한 루틴에서만 의미가 있어 루틴이 길고 복잡해지기 쉽습니다. 따라서 다음으로 할 리팩토링은 이런 변수들을 제거하는 것입니다. 그중에서 `format`이 가장 만만해 보입니다. `format`은 임시 변수에 함수를 대입한 형태인데, 필자는 함수를 직접 선언해 사용하도록 바꾸는 편입니다.

<br>

```js
// statement() 함수...
function format(aNumber) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format(aNumber);
}
```

<br>

```js
// 최상위...
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `청구 내역 (고객명: ${invoice.customer})\n`;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);

    // 청구 내역을 출력한다.
    result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${
      perf.audience
    }석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount / 100)}\n`; // 임시 변수였던 format을 함수 호출로 대체
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}
```

<br>

그러나 `format`은 이 함수가 하는 일을 충분히 설명해주지 못합니다. 이 함수의 핵심은 화폐 단위 맞추기입니다. 그런 느낌을 살리는 이름을 골라서 다음과 같이 `함수 선언 바꾸기`를 적용해봅시다.

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;
    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);

      // 청구 내역을 출력한다.
      // result += `  ${playFor(perf).name}: ${format(amountFor(perf) / 100)} (${perf.audience}석)\n`;
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`; // 함수 이름 변경, /100 삭제
      totalAmount += amountFor(perf);
    }
    // result += `총액: ${format(totalAmount/100)}\n`;
    result += `총액: ${usd(totalAmount)}\n`; // 함수 이름 변경, /100 삭제
    result += `적립 포인트: ${volumeCredits}점\n`;
    return result;
  }
```

<br>

```js
// statement() 함수...
// function format(aNumber) {
function usd(aNumber) {
  // 함수 이름 변경
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    minimumFractionDigits: 2,
  }).format(aNumber / 100); // 단위 변환 로직도 이 함수 안으로 이동
}
```

<br>

긴 함수를 쪼개는 리팩터링은 이름을 잘 지어야만 효과가 있습니다. 이름이 좋으면 함수 본문을 읽지 않고도 무슨 일을 하는지 알 수 있습니다. 단번에 좋은 이름을 짓기는 쉽지 않으니 처음에는 당장 떠오르는 최선의 이름을 사용하다가, 나중에 더 좋은 이름이 떠오를 때 바꾸는 식이 좋습니다. 앞서 이름을 바꿀 때, 여러 차례 등장하는 100으로 나누는 코드도 추출한 함수로 옮겼습니다. 화면에 출력할 때는 다시 달러 단위로 변환해야 하므로 포맷 변환 함수인 `usd()`에서 나눗셈까지 처리해주면 좋습니다.

<br>

### `volumeCredits` 변수 제거하기

<br>

이 변수는 반복문을 한 바퀴 돌 때마다 값을 누적하기 때문에 리팩터링하기가 더 까다롭습니다. 따라서 `반복문 쪼개기`로 값이 누적되는 부분을 따로 뺴냅니다.

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let totalAmount = 0;
    let volumeCredits = 0;
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;

    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);

      // 청구 내역을 출력한다.
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
      totalAmount += amountFor(perf);
    }

    for (let perf of invoice.performances) { // 값 누적 로직을 별도 for문으로 분리
      volumeCredits += volumeCreditsFor(perf);
    }

    result += `총액: ${usd(totalAmount)}\n`;
    result += `적립 포인트: ${volumeCredits}점\n`;
    return result;
  }
```

<br>

이어서 `문장 슬라이드하기`를 적용해서 `volumeCredits` 변수를 선언하는 문장을 바로 앞으로 옮깁니다.

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let totalAmount = 0;
    // let volumeCredits = 0;
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;

    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);

      // 청구 내역을 출력한다.
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
      totalAmount += amountFor(perf);
    }
    let volumeCredits = 0; // 변수 선언(초기화)을 반복문 앞으로 이동
    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);
    }

    result += `총액: ${usd(totalAmount)}\n`;
    result += `적립 포인트: ${volumeCredits}점\n`;
    return result;
  }
```

<br>

`volumeCredits` 값 갱신과 관련한 문장들을 한데 모아두면 `임시 변수를 질의 함수로 바꾸기`가 수월해집니다. 이번에도 역시 `volumeCredits` 값 계산 코드를 `함수로 추출`하는 작업부터 합니다.

<br>

```js
// statement() 함수...
function totalVolumeCredits() {
  // 함수로 추출
  let volumeCredits = 0;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  return volumeCredits;
}
```

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let totalAmount = 0;
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;

    for (let perf of invoice.performances) {
      volumeCredits += volumeCreditsFor(perf);

      // 청구 내역 출력하기
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
      totalAmount += amountFor(perf);
    }
    let volumeCredits = totalVolumeCredits(); // 값 계산 로직을 함수로 추출

    result += `총액: ${usd(totalAmount)}\n`;
    result += `적립 포인트: ${volumeCredits}점\n`;
    return result;
  }
```

<br>

`함수 추출`이 끝났다면, 다음은 `volumeCredits` `변수를 인라인`할 차례입니다.

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let totalAmount = 0;
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;

    for (let perf of invoice.performances) {

      // 청구 내역을 출력한다.
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
      totalAmount += amountFor(perf);
    }

    result += `총액: ${usd(totalAmount)}\n`; // 변수 인라인
    result += `적립 포인트: ${totalVolumeCredits()}점\n`; // volumeCredits => totalVolumeCredits()
    return result;
  }
```

<br>

반복문을 쪼개서 성능이 느려지지 않을까 걱정할 수 있습니다. 이처럼 반복문이 중복되는 것을 꺼리는 이들이 많지만, 이정도 중복은 성능에 미치는 영향이 미미할 때가 많습니다. 때로는 리팩터링이 성능에 상당한 영향을 주기도 합니다. 그런 경우라도 개의치 않고 리팩터링하는 이유는 잘 다듬어진 코드라야 성능 개선 작업도 훨씬 수월하기 때문입니다. 리팩터링 과정에서 성능이 크게 떨어졌다면 리팩터링 후 시간을 내여 성능을 개선합니다. 이 과정에서 리팩터링된 코드를 예전으로 되돌리는 경우도 있지만, 대체로 리팩털이 덕분에 성능 개선을 더 효과적으로 수행할 수 있습니다. 결과적으로 더 깔끔하면서 더 빠른 코드를 얻게 됩니다.

<br>

`volumeCredits` 변수를 제거하는 작업의 단계를 아주 잘게 나눴다는 점에도 주목합시다. 다음과 같이 총 네 단계로 수행했으며, 각 단계마다 `컴파일-테스트`하고 로컬 저장소에 커밋했습니다.

<br>

1.`반복문 쪼개기`로 변수 값을 누적시키는 부분을 분리합니다. 2. `문장 슬라이드하기`로 변수 초기화 문장을 변수 값 누적 코드 바로 앞으로 옮깁니다. 3. `함수 추출하기`로 적립 포인트 계산 부분을 별도 함수로 추출합니다. 4. `변수 인라인하기`로 `volumeCredits` 변수를 제거합니다.

<br>

코드가 복잡할수록 단계를 작게 나누면 작업속도가 빨라집니다. 그래서 필자는 상황이 복잡해지면 단계를 더 작게 나누는 일을 가장 먼저 합니다.

<br>

`totalAmount`도 앞에서와 똑같은 절차로 제거합니다. 먼저 반복문을 쪼개고, 변수 초기화 문장을 옮긴 다음, 함수를 추출합니다. 여기서 한 가지 문제는 추출할 함수의 이름으로는 `totalAmount`가 가장 좋지만 이미 같은 이름의 변수가 있어서 쓸 수 없습니다. 그래서 일단 아무 이름인 `appleSauce`를 붙여주겠습니다. (`컴파일-테스트-커밋`)

<br>

```js
// statement() 함수...
function appleSauce() {
  // 함수 추출하기
  let totalAmount = 0;
  for (let perf of invoice.performance) {
    totalAmount += amountFor(perf);
  }
  return totalAmount;
}
```

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;
    for (let perf of invoice.performances) {
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
    }
    let totalAmount = appleSauce(); // 함수 추출 & 임시 이름 부여

    result += `총액: ${usd(totalAmount)}\n`;
    result += `적립 포인트: ${totalVolumeCredits()}점\n`;
    return result;
  }
```

<br>

이제 `totalAmount` 변수를 인라인한 다음 (`컴파일-테스트-커밋`), 함수 이름을 더 의미 있게 고칩니다. (`컴파일-테스트-커밋`)

<br>

```js
// 최상위...
  function statement(invoice, plays) {
    let result = `청구 내역 (고객명: ${invoice.customer})\n`;
    for (let perf of invoice.performances) {
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf)} (${perf.audience}석)\n`;
    }

    // let totalAmount = appleSauce();

    result += `총액: ${usd(totalAmount())}\n`; // 변수 인라인 후 함수 이름 바꾸기
    result += `적립 포인트: ${totalVolumeCredits()}점\n`;
    return result;
  }
```

<br>

```js
// statement() 함수...
// function appleSauce() { // 함수 추출하기
function totalAmount() {
  //  함수 이름 바꾸기
  let totalAmount = 0;
  for (let perf of invoice.performance) {
    totalAmount += amountFor(perf);
  }
  return totalAmount;
}
```

<br>

추출한 함수 안에서 쓰인 이름들도 코딩 스타일에 맞게 변경합니다.

<br>

```js
// statement() 함수...
function totalAmount() {
  let result = 0; // 변수 이름 바꾸기
  for (let perf of invoice.performance) {
    result += amountFor(perf); //
  }
  return result; //
}

function totalVolumeCredits() {
  let result = 0; // 변수 이름 바꾸기
  for (let perf of invoice.performances) {
    result += volumeCreditsFor(perf); //
  }
  return result; //
}
```

<br>

<br>

## 다음 챕터

- [1.5 - 중간 점검: 난무하는 중첩 함수](https://github.com/Esoolgnah/Summary_of_Refactoring_2nd_Edition/blob/main/01_리팩터링_첫번째_예시/01_05_중간_점검:난무하는_중첩_함수.md)

<br>

## 이전 챕터

- [1.3 - 리팩터링의 첫 단계](https://github.com/Esoolgnah/Summary_of_Refactoring_2nd_Edition/blob/main/01_리팩터링_첫번째_예시/01_03_리팩터링의_첫_단계.md)

<br>

## 목록으로

- [목록](https://github.com/Esoolgnah/Summary_of_Refactoring_2nd_Edition/blob/main/01_리팩터링_첫번째_예시/01_00_리팩터링_첫번째_예시.md)
