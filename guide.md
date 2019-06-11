# 시작하기
---

## HTML
> 사용자가 로딩 없이 바로 볼 수 있도록 첫 페이지는 서버에서 그려줍니다. <br> id는 변경 가능하며 리스트 되는 부분에 data-page 속성을 추가 합니다.

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
> 이미지 FOUC 방지를 위해 높이를 지정이 필요합니다.

```css
/* 이미지 높이 FOUC 대응 */
.hwd_grid_lst .pd_unit .thmb{position:relative;display:block;padding-bottom:100%}
.hwd_grid_lst .pd_unit .thmb:before{content:'';position:absolute;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.03)}
.hwd_grid_lst .pd_unit .thmb img{position:absolute;top:0;left:0;right:0;bottom:0;max-width:100%;max-height:100%;object-fit:cover}

/* 로딩바 */
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
> SSG-InfiniteGrid를 적용하기 위해서 상황에 맞게 아래 스크립트 파일을 로드 합니다.

#### ssg.common.infinitegrid.js
> 다음 스크립트 파일은 범용적으로 사용 가능합니다.
* 하우디 메인 NEW ARRIVALS
* 모바일 특가매장 (오반장, 해바)

```html
PC
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/ssg/js/ui/ssg.common.infinitegrid.js"></script>
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.common.infinitegrid.js"></script>
```

#### ssg.view.infinitecategory.js
> 다음은 탭메뉴 형식의 UI 에서 무한 스크롤을 사용합니다.
* 공식스토어 메인 카테고리별 인기상품
* 새벽배송 신선식탁

```html
PC
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/ssg/js/ui/ssg.view.infinitecategory.js"></script>
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.view.infinitecategory.js"></script>
```


#### ssg.view.infinitegrid.js
> 공통 상품유닛 무한 스크롤 대응을 위해 사용합니다.
* 모바일 대카테고리

```html
MO
<script type="text/javascript" src="http://ui.ssgcdn.com/ui/m_ssg/js/ui-renew/ssg.view.infinitegrid.js"></script>
```

### Initialize
```js
$(function(){
    var oInfiniteGrid = new ssg.View.infiniteGrid({
        nEndPage: 30 // 마지막 데이터 페이지
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

#### 임시 데이터 처리
```html
<script id="_tplGridItemTest" type="text/x-jquery-tmpl">
<li data-page="{{= page}}" class="shd_sec shdu_sec_item">
	<div class="shdu_item">
        // ...
	</div>
</li>
</script>
```
