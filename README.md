# virtual-dom

### 왜 있는가?

- 수많은 데이터가 있는 웹의 경우, DOM에 직접 접근하여 변화를 주면 느려짐
- DOM 자체를 읽고 쓰는 성능은 자바스크립트 객체를 처리할 때의 성능과 비슷
- 웹 브라우저 단에서 DOM에 변화가 일어나면 웹 브라우저가 CSS 를 다시 연산, 레이아웃 구성, 리페인트 → 시간이 허비됨
- **DOM 을 최소한으로 조작하자!**
- **DOM 업데이트를 추상화 → DOM 처리 횟수를 최소화**

### Vitual DOM

- 실제 돔을 조작하지 않고, 이를 추상화한 자바스크립트 객체를 구성하여 사용 (실제 돔의 가벼운 사본)
- 리액트 엔진이 이전 가상돔의 스냅샷을 찍어둔 것과 새로운 가상돔과 비교해서 변경된 부분만 실제 DOM에 적용
- 변경된 부분은 묶어서 한번에 업데이트(batch update)
    
    <aside>
    💡 Batch update
    - 리액트 성능을 위해 setState를 단일 업데이트로 한번에 처리하는 것 
    - 현재는 이벤트 **핸들러의 업데이트에만 기본적으로 적용됨 
    - 하지만 이벤트 핸들러가 비동기로 실행될 경우, 독립된 state의 값 업데이틑 일괄적으로 처리되지 않음**
    
    </aside>
    

### 재조정 Reconciliation

- O(n) 복잡도의 휴리스틱 알고리즘
    - 서로 다른 두 엘리먼트는 서로 다른 트리를 만들어 낸다
    - 개발자가 `key` prop을 통해, 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해줄 수 있다.
- 비교 알고리즘
    - 두 개의 트리를 비교할 때 React는 두 엘리먼트의 root 엘리먼트부터 비교함
        - 루트 엘리먼트 타입에 따라 달라짐
        - 엘리먼트 타입이 다르다면?
            - **React는 이전 트리를 버리고 완전히 새로운 트리를 구축**
            ```jsx
            <a> → <img>, <Article> → <Comment>, <Button> →<div>
            ```
             - 트리를 버릴 때 이전 DOM 노드는 모두 파괴됨
            - 새로운 트리가 만들어질 때 새로운 DOM노드들이 DOM에 삽입됨
            - 이전 트리와 연관된 모든 state는 사라짐
        
        ```jsx
        <div>
          <Counter />
        </div>
        
        <span>
          <Counter />
        </span>
        
        // 이전 <Counter />는 사라지고, 그 state도 사라짐 , 새로 다시 마운트됨
        ```
        
        - 엘리먼트의 타입이 같다면?
            - 두 엘리먼트의 속성을 확인하여 변경된 속성들만 갱신함
            
            ```jsx
            <div className="before" title="stuff" />
            
            <div className="after" title="stuff" />
            
            // className만 수정함
            
            <div style={{color: 'red', fontWeight: 'bold'}} />
            
            <div style={{color: 'green', fontWeight: 'bold'}} />
            // color만 수정 
            ```
            
            - 컴포넌트가 갱신되면 인스턴스는 동일하게 유지 → state가 유지됨
            - 새로운 엘리먼트 내용으 반영하기 위해 현재 컴포넌트 인스턴스의 props를 갱신함
    - **자식에 대한 재귀적 처리**
        
        ```jsx
        <ul>
          <li>first</li>
          <li>second</li>
        </ul>
        
        <ul>
          <li>first</li>
          <li>second</li>
          <li>third</li>
        </ul>
        
        // 아래처럼 리스트 맨 앞에 추가하는 것은 성능이 좋지 않음 
        
        <ul>
          <li>Duke</li>
          <li>Villanova</li>
        </ul>
        
        <ul>
          <li>Connecticut</li>
          <li>Duke</li>
          <li>Villanova</li>
        </ul>
        
        //  <li>Duke</li>, <li>Villanova</li> 종속 트리를 그대로 유지하지 않고 모든 자식을 변경하기 때문 
        ```
        
        - Keys
            - 위의 이러한 비효율을 해결하기 위해 `key` 속성 지원
            - 자식들이 key를 가지고 있다면 key를 통해 전후 트리의 자식들이 일치하는지 확인
            - 형제 사이에서만 유일하면 되고, 전역에서 유일할 필요는 없음
                - 최후의 수단으로 index 사용 가능함 (재배열되지 않는 경우)
                - 변하는 key(`Math.random()`으로 생성된 값 등)를 사용하면 컴포넌트 인스턴스와 DOM 노드를 불필요하게 재생성하여 성능이 나빠지거나 자식 컴포넌트의 state가 유실될 수 있음
            
            ```jsx
            <ul>
              <li key="2015">Duke</li>
              <li key="2016">Villanova</li>
            </ul>
            
            <ul>
              <li key="2014">Connecticut</li>
              <li key="2015">Duke</li>
              <li key="2016">Villanova</li>
            </ul>
            ```
            

### 오해

- 가상 돔을 사용한다고 무조건 빨라지는 것은 아님
- “지속적으로 데이터가 변화하는 대규모 어플리케이션 구축하기”에 적합
- 단순 라우팅만 있는 정적 페이지는 사용하지 않는 것이 좋다


### Reference
https://ko.reactjs.org/docs/reconciliation.html#gatsby-focus-wrapper
