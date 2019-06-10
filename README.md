# Get Started
# 시작하기
## HTML

첫 페이지 부분은 서버에서 그려줍니다.
```html
<div id="_infinite_grid">
	<ul>
		<li data-page="1">
		    <div>test1</div>
		</li>
		<li data-page="1">
		    <div>test2</div>
		</li>
		// ...
	</ul>
	<div class="grid_loader"><div class="grid_loading"><div class="dot1"></div><div class="dot2"></div><div class="dot3"></div></div></div>
</div>
```

## CSS
```css
/* infinite grid reset */
.hwd_grid_lst .pd_unit .thmb{position:relative;display:block;padding-bottom:100%}
.hwd_grid_lst .pd_unit .thmb:before{content:'';position:absolute;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.03)}
.hwd_grid_lst .pd_unit .thmb img{position:absolute;top:0;left:0;right:0;bottom:0;max-width:100%;max-height:100%;object-fit:cover}

.grid_loader{display:none}
.grid_loader.on{display:block}
.grid_loading{display:flex;justify-content:center;align-items:center;height:80px;overflow:hidden}
.grid_loading>div{width:10px;height:10px;background-color:#222;border-radius:100%;display:inline-block;margin:0 3px;-webkit-animation:bouncedelay 1.4s infinite ease-in-out both;animation:bouncedelay 1.4s infinite ease-in-out both}
.grid_loading .dot1{-webkit-animation-delay:-0.32s;animation-delay:-0.32s}
.grid_loading .dot2{-webkit-animation-delay:-0.16s;animation-delay:-0.16s}
@-webkit-keyframes bouncedelay {
	0%, 80%, 100% {-webkit-transform: scale(0)}
	40% {-webkit-transform: scale(1.0)}
}
@keyframes bouncedelay {
	0%, 80%, 100% {-webkit-transform: scale(0);transform: scale(0)}
	40% {-webkit-transform: scale(1.0);transform: scale(1.0)}
}
```

## JS
### import javascript files
#### ssg.common.infinitegrid.js (범용적으로 사용)
```html
PC
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/ssg/js/ui/ssg.common.infinitegrid.js"></script>
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.common.infinitegrid.js"></script>
```


#### ssg.view.infinitecategory.js (페이지 네비게이션 타입)
```html
PC
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/ssg/js/ui/ssg.view.infinitecategory.js"></script>
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.view.infinitecategory.js"></script>
```


#### ssg.view.infinitegrid.js (공통 상품유닛)
```html
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.view.infinitegrid.js"></script>
```

### Initialize
```js
$(function(){
    var oInfiniteGrid = new ssg.View.infiniteGrid({
        nEndPage: 30 // 마지막 데이터 페이지
        // Custom options
        // elGrid: '#_infinite_grid',
        // elLoading: '.grid_loader',
        // elFooter: '#mcom_footer',
        // egOptions: {
        //     useRecycle: false, // DOM의 수를 유지 여부
        //     isEqualSize: false // 모든 요소의 크기가 서로 같은지 여부
        // }
    });

    // 상품추가
    oInfiniteGrid.on('fetchItems', function(nCurPage) {
        var oSelf = this;
        // 임시 데이터 처리
        var getItems = function(length) {
            var arr = [];
            for (var i = 0; i < length; ++i) {
                arr.push({
                    no: i,
                    page: nCurPage,
                    title: (i + 1) + " 상품명이름1234567890r가나다라마바사~@#$%^&*()_+",
                    items: new Array(12)
                });
            }
            return arr;
        };
        var aDummyData = $('#_tplGridItemTest').tmpl(getItems(30));
        this.addItemList(aDummyData);

        /* 개발 참고
        $.ajax({

        }).done(function(data) {
            var aItems = $($.parseHTML(data)).filter('li.shdu_sec_item');
            oSelf.addItemList(aItems);
        }).fail(function() {
            // 데이터 처리 실패 시
            oSelf.lastRander();
        });
        */
    });

    // 뒤로가기
    var oInfinitePersist = oInfiniteGrid.oPersist;
    if (oInfinitePersist.get('') !== null) {
        oInfiniteGrid.historyBack();
    } else {
        oInfiniteGrid.firstRender();
    }
});
```

#### 임시 데이터 처리 템플릿
```html
<script id="_tplGridItemTest" type="text/x-jquery-tmpl">
<li data-page="{{= page}}" class="shd_sec shdu_sec_item">
	<div class="shdu_item">
        // ...
	</div>
</li>
</script>
```



# 코드 리뷰
## 무한 스크롤 적용
* [egjs-infinitegrid options](https://naver.github.io/egjs-infinitegrid/#options)
* [egjs-infinitegrid API](https://naver.github.io/egjs-infinitegrid/release/latest/doc/)


자연스러운 무한 스크롤을 위해서 하단 footer부분은 숨겨놓습니다.

```js
_assignComponents: function(){
    if (this._welGridItems.length) {
        this._hideBottom();
        this._infiniteGrid();
    }
    // ...
},
_hideBottom: function(){
    this._welFooter.hide();
},
```

eg.InfiniteGrid 객체 생성

```js
_infiniteGrid: function(){
    this.oGrid = new eg.InfiniteGrid(this._welGridList.get(0), $.extend({}, this.egDefaults, this.egOptions));
    // 레이아웃 초기화
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
_onLayoutComplete: function(e){
    // ...
    if (this.isFirstRander) {
        this.isFirstRander = false;
        this._welGridItems.remove();
    }

    if (this.isLoading && !e.fromCache) {
        this._endLoding();
    }
},
```

더이상 불러올 데이터가 없으면 숨겨놓은 footer부분을 보여줍니다.
```js
_onAppend: function(e){
    // ...
    if (nPage >= this.nEndPage) {
        this.lastRander();
        return;
    }
    this._startLoding();
    this._bAppend = true;
    this.nCurPage = nPage + 1;
    this.emit('fetchItems', this.nCurPage);
},
lastRander: function(){
    this._endLoding();
    this._showBottom();
},
```


## 이미지 레이지 로딩

IntersectionObserver 객체 생성

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

요청받은 데이터에 빈 이미지 셋팅

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

상품이 추가되고 레이아웃 배치가 완료 되면 원본 이미지 로딩 처리

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

#### eg.Persist 객체 생성
```js
_assignComponents: function(){
    // ...
    this.oPersist = new eg.Persist('listModule');
},
```

#### 상품 클릭 페이지 이동 시 현재 데이터 담기
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

#### 뒤로가기 저장된 데이터 확인
```js
var oInfinitePersist = oInfiniteGrid.oPersist;

if (oInfinitePersist.get('') !== null) {
    oInfiniteGrid.historyBack();
} else {
    oInfiniteGrid.firstRender();
}
```

#### 뒤로가기 처리
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

```js
_onLayoutComplete: function(e){
    // ...
    if (this.isHistoryBack) {
        this.isHistoryBack = false;
        this._welHtmlBody.animate({scrollTop: this.nCurScroll}, 250);
        this.oGrid.layout(true);
        this._delayRender();
    }
    // ...
},
_delayRender: function(){
    this._welGrid.one('touchstart', function(){
        this.isRendering = false;
    }.bind(this));
},
```
