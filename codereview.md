
# 코드 리뷰
---

## 초기화

> 사용자 정의 옵션

```js
var oInfiniteGrid = new ssg.View.infiniteGrid({
    nEndPage: 30 // 마지막 데이터 페이지
    elGrid: '#_infinite_grid', // wrapper id
    elList: '>ul',
    elItem: '>li',
    elLoading: '.grid_loader',
    elFooter: '#mcom_footer',
    egOptions: {
        useRecycle: true, // DOM의 수를 유지 여부
        isEqualSize: true // 모든 요소의 크기가 서로 같은지 여부
    }
});
```

> 반복적으로 사용되는 DOM 캐싱, EventHandlers 등록

```js
init: function(){
    this._assignElements();
    this._assignComponents();
    this._attachEventHandlers();
},
_assignElements: function(){
    this._window = $(window);
    this._welBody = $('body');
    this._welGrid = $(this.elGrid);
    this._welGridList = this._welGrid.find(this.elList);
    this._welGridItems = this._welGridList.find(this.elItem);
    this._welGridLoading = this._welGrid.find(this.elLoading);
    this._welFooter = $(this.elFooter);
},
_attachEventHandlers: function() {
    this._welGridList.on('click', '>li a', $.proxy(this._onClickItemList, this));
    this._window.on('orientationchange', $.proxy(this._onOrientationChange, this));
},
```

## 무한 스크롤 적용

> 자연스러운 무한 스크롤을 위해서 하단 footer부분은 숨겨놓습니다.

```js
_assignComponents: function(){
    if (this._welGridItems.length) {
        this._hideBottom();
        this._infiniteGrid();
        this._lazyLoader();
    }
    this.oPersist = new eg.Persist('listModule');
},
_hideBottom: function(){
    this._welFooter.hide();
},
```

> eg.InfiniteGrid 객체 생성

```js
_infiniteGrid: function(){
    // 초기화
    this.oGrid = new eg.InfiniteGrid(
        this._welGridList.get(0),
        $.extend({}, this.egDefaults, this.egOptions)
    );

    // GridLayout 셋팅
    this.oGrid.setLayout(eg.InfiniteGrid.GridLayout);
    // 위로 상품 추가
    this.oGrid.on('prepend', $.proxy(this._onPrepend, this));
    // 아래로 상품 추가
    this.oGrid.on('append', $.proxy(this._onAppend, this));
    // 스크롤 이벤트
    this.oGrid.on('change', $.proxy(this._onChange, this));
    // 상품 추가, 레이아웃 배치 완료 후
    this.oGrid.on('layoutComplete', $.proxy(this._onLayoutComplete, this));
},
```

> append(), prepend(), layout() 메서드 호출 후 레이아웃 배치가 완료됐을 때 발생하는 이벤트

```js
_onLayoutComplete: function(e){
    e.target.forEach(function(item) {
        var elem = item.el;
        if (!elem) { return; }

        if (this.observer) {
            this.observer.observe(elem);
        } else {
            this._loadImages(elem);
        }
    }.bind(this));

    if (this.isFirstRander) {
        this.isFirstRander = false;
        this._welGridItems.remove();
    }

    if (this.isHistoryBack) {
        this.isHistoryBack = false;
        this._delayRender().then(function(){
            this._welHtmlBody.animate({scrollTop: this.nCurScroll}, 250, function(){
                this.oGrid.layout(true);
            }.bind(this));
        }.bind(this));
    }

    if (this.isLoading && !e.fromCache) {
        this._endLoding();
    }
},
```

> 처음 레이아웃 렌더링 시 기존 마크업 요소 복사 후 append

```js
firstRender: function(){
    if (!this.oGrid) { return; }
    var aItems = $.map(this._welGridItems.clone(), function(val) {
        return val;
    });
    this.isFirstRander = true;
    this.nCurPage = 1;
    this.oGrid.append(aItems, this.nCurPage);
},
```

> 레이아웃 위에 데이터를 추가 할 때 발생하는 이벤트

```js
_onPrepend: function(e){
    var groupKeys;
    var nPage;

    if (this.isLoading || this.isRendering) {
        return;
    }

    groupKeys = this.oGrid.getGroupKeys(true);
    nPage = groupKeys[0] || e.groupKey;

    if (nPage <= 1) {
        return;
    }
    this._startLoding();
    this._bAppend = false;
    this.nCurPage = nPage - 1;
    this.emit('fetchItems', this.nCurPage);
},
```

> 레이아웃 아래에 데이터를 추가 할 때 발생하는 이벤트

```js
_onAppend: function(e){
    var groupKeys;
    var nPage;

    if (this.isLoading || this.isRendering) {
        return;
    }

    groupKeys = this.oGrid.getGroupKeys(true);
    nPage = groupKeys[groupKeys.length - 1] || e.groupKey;

    if (nPage >= this.nEndPage) {
        this.lastRander();
        return;
    }
    this._startLoding();
    this._bAppend = true;
    this.nCurPage = nPage + 1;
    this.emit('fetchItems', this.nCurPage);
},
```

> 더이상 불러올 데이터가 없으면 숨겨놓은 footer부분을 보여줍니다.

```js
lastRander: function(){
    this._endLoding();
    this._showBottom();
},
```


## 이미지 레이지 로딩

> IntersectionObserver 객체 생성

```js
_lazyLoader: function(){
    var oSelf = this;
    var config = {
        rootMargin: '200px 0px 200px 0px',
        threshold: 0
    };

    if (!('IntersectionObserver' in window)) {
        this.observer = null;
        return;
    }

    this.observer = new IntersectionObserver(function(entries, observer) {
        entries.forEach(function(entry) {
            if (!entry.isIntersecting) {
                return;
            }
            var elem = entry.target;
            oSelf.observer.unobserve(elem);
            oSelf._loadImages(elem);
        });
    }, config);
},
```

> 요청받은 데이터에 빈 이미지 셋팅

```js
addItemList: function(aItemList){
    var aItems = $.each(aItemList, function(i, val) {
        var welTarget = $(val);
        var welImages = welTarget.find('a img');
        welImages.each(function(){
            var image = $(this);
            var orgSrc = image.attr('src');
                image.attr({
                'src': 'http://ui.ssgcdn.com/ui/ssg/img/common/b.gif',
                'data-src': orgSrc
            }).addClass('lazy_img');
        });
    });
    this.oGrid[this._bAppend ? 'append' : 'prepend'](aItems, this.nCurPage);
},
```

> 데에터가 추가되고 레이아웃 배치가 완료 되면 IntersectionObserver 지원 여부에 따른 이미지 처리

```js
_onLayoutComplete: function(e){
    e.target.forEach(function(item) {
        var elem = item.el;
        if (!elem) { return; }

        if (this.observer) {
            this.observer.observe(elem);
        } else {
            this._loadImages(elem);
        }
    }.bind(this));
    // ...
},
```

> 원본 이미지 로드

```js
_loadImages: function(elem){
    var lazyImages = elem.querySelectorAll('.lazy_img');

    if (!lazyImages.length) {
        return;
    }
    lazyImages.forEach(function(image){
        if (image.classList.contains('loaded')) {
            return;
        }
        image.src = image.dataset.src;
        image.classList.add('loaded');
    });
},
```



## 뒤로가기 이전 상태 유지

> eg.Persist 객체 생성

```js
_assignComponents: function(){
    // ...
    this.oPersist = new eg.Persist('listModule');
},
```

>  상품 클릭 시 (페이지 이동) 현재 데이터 담기

```js
_attachEventHandlers: function() {
    this._welGridList.on('click', '>li a', $.proxy(this._onClickItemList, this));
    // ..
},
_onClickItemList: function(e){
    var welTarget = $(e.currentTarget);
    var welCurrentItem = welTarget.closest('li');

    var nCurrentKey = welCurrentItem.data('groupkey');
    var nStart = nCurrentKey - 1;
    var nEnd = nCurrentKey + 1;

    var groupKeys = this.oGrid.getGroupKeys(true);
    var nFirstKey = groupKeys[0];
    var nLastKey = groupKeys[groupKeys.length - 1];

    if (this.oGrid.isProcessing()) {
        e.preventDefault();
    }

    nStart = (nStart >= nFirstKey) ? nStart : nFirstKey;
    nEnd = (nEnd <= nLastKey) ? nEnd : nLastKey;

    this.oPersist
    .set('curItems', this.oGrid.getStatus(nStart, nEnd))
    .set('curPage', welCurrentItem.data('page'))
    .set('curScroll', window.scrollY);
}
```

> 뒤로가기 시 저장된 데이터 확인

```js
var oInfinitePersist = oInfiniteGrid.oPersist;

if (oInfinitePersist.get('') !== null) {
    oInfiniteGrid.historyBack();
} else {
    oInfiniteGrid.firstRender();
}
```

> 이전 레이아웃 데이터 복원

```js
historyBack: function(){
    if (!this.oGrid) {  return; }
    this.isRendering = true;
    this.isHistoryBack = true;
    this.nCurPage = this.oPersist.get('curPage');
    this.nCurScroll = this.oPersist.get('curScroll');
    this.oGrid.clear();
    this.oGrid.setStatus(this.oPersist.get('curItems'));
    if (this._isEndPage()) {
        this.lastRander();
    }
    eg.Persist.clear();
},
_onPrepend: function(e){
    // ...
    if (this.isLoading || this.isRendering) {
        return;
    }
    // ...
},
_onAppend: function(e){
    // ...
    if (this.isLoading || this.isRendering) {
        return;
    }
    // ...
},

```
> 레이아웃이 배치가 끝나면 dom complete 상태가 될때까지 기다린 후 이전 스크롤로 이동

```js
_onLayoutComplete: function(e){
    // ...
    if (this.isHistoryBack) {
        this.isHistoryBack = false;
        this._delayRender().then(function(){
            this._welHtmlBody.animate({scrollTop: this.nCurScroll}, 250, function(){
                console.log('scroll');
                this.oGrid.layout(true);
            }.bind(this));
        }.bind(this));
    }
    // ...
},
_delayRender: function(){
    var oSelf = this;
    var fnInterval = null;
    return new Promise(function(resolve, reject) {
        if (document.readyState === 'complete') {
            resolve();
        } else {
            fnInterval = setInterval(function() {
                console.log(document.readyState);
                if(document.readyState === 'complete') {
                    clearInterval(fnInterval);
                    resolve();
                }
            }, 100);
        }
    }).then(function(){
        oSelf._welGrid.one('touchstart', function(){
            oSelf.isRendering = false;
        });
    });
},
```