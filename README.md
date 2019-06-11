## SSG-InfiniteGrid
> 레이아웃 유형에 따라 내용을 포함하여 카드 요소를 무한대로 정렬하는 데 사용되는 모듈입니다. 이 모듈을 사용하면 크기가 다른 여러 카드 요소로 구성된 다양한 레이아웃을 구현할 수 있습니다. 어떤 상황에서도 모듈이 처리하는 DOM의 수를 유지함으로써 성능을 보장합니다


## egjs
> SSG-InfiniteGrid는 [egjs](https://naver.github.io/egjs/) 라이브러리를 기반으로 작업되었습니다. <br> egjs는 웹 애플리케이션을 가장 쉽고 빠르게 작성할 수있는 Javascript 구성 요소 그룹

- **InfiniteGrid (v3.5.3)** 레이아웃 유형에 따라 내용을 포함하여 카드 요소를 무한대로 정렬하는데 사용되는 모듈
    - [egjs-infinitegrid Options](https://naver.github.io/egjs-infinitegrid/#options)
    - [egjs-infinitegrid API](https://naver.github.io/egjs-infinitegrid/release/latest/doc/)

- **Persist (v2.2.1)** 히스토리 탐색 중에 지속 된 데이터를 처리하기위한 캐시 인터페이스를 제공

## 기능

- DOM이 계속 쌓이는 형태 → 화면에 보이는 DOM만 유지 (화면에 보이지 않는 요소는 제거)
- 일정한 DOM의 수를 유지함으로써 성능향상 기대
- 위(prepend), 아래(append) 모두 데이터를 불러 올 수 있는 UI인터렉션
- 상품 클릭 페이지 이동 시 현재 데이터 기억
- 뒤로가기(히스토리백) 시 이전 데이터 상태 유지

## 예제

## Browser support
<table>
  <thead>
    <tr>
      <th>Internext Explorer</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Safari</th>
      <th>IOS</th>
      <th>Android</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>8+ (possibly 9 also)</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>Yes</td>
      <td>7+</td>
      <td>2.1+ (except 3.x)</td>
    </tr>
  </tbody>
</table>