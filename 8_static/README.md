# IBSheet 객체 사용하기

## static 객체란
시트 생성시 [IBSheet](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/static/static).[create](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/static/create) 라는 함수를 통해 생성이 되었는데, 이때 사용된 IBSheet가 static 객체 입니다.<br>
이 객체는 ibsheet.js 파일을 임포트하면 글로벌 객체로 자동 생성 됩니다.<br>
이 객체를 이용하면 시트 생성 외에도 모든 시트에 동일한 기능을 적용하거나 시트의 Calendar객체를 시트 외부에서 사용하는 등을 다양한 기능이 제공됩니다.

## 모든 시트에 공통 기능 정의 하기1 (CommonOptions)
IBSheet.CommonOptions 속성을 통해 모든 시트에 동일한 기능을 적용할 수 있습니다.<br>
해당 기능은 plugins/ibsheet-common.js 에 기본적으로 정의 되어 있습니다.
```js
IBSheet.CommonOptions = {
    Cfg:{
        //엑셀,text 다운로드 서버 모듈 URL 정의
        Export:{
            Down2ExcelUrl: "/sheet/jsp/Down2Excel.jsp",
            Down2TextlUrl: "/sheet/jsp/Down2Text.jsp" 
        }
    },
    Def:{
        Header: {
            // 모든 시트의 헤더행의 배경색을 회색으로
            Color: "#DCDCDC"
        }
    }
}
```
## 모든 시트에 공통 기능 정의 하기2 (onBeforeCreate)
IBSheet.create 함수를 통해 시트가 생성될 때 onBeforeCreate 이벤트가 발생합니다.<br>
이 이벤트를 통해 각 화면에서 구성한 시트 초기화 로직을 수정할 수 있습니다.
```js
IBSheet.onBeforeCreate = function(init) {
    function floatFormat(cols){
        if(cols && cols.length>0){
            cols.forEach((col,idx)=>{
                // 실수 타입의 경우 소수점 둘째자리까지 표현되게 끔 수정
                if( col.Type == "Float" ){
                    col.Format = "#,##0.##";
                }
            });
        }
        return cols;
    }
    floatFormat(init?.options?.LeftCols);
    floatFormat(init?.options?.Cols);
    floatFormat(init?.options?.RightCols);
    return init; //반드시 리턴해야 반영됩니다.
}
```


## 외부에서 시트 달력 사용하기
시트가 갖고 있는 달력객체를 외부(html input element)에서 사용하실 수 있습니다.

사용 방법은 다음과 같습니다.
```html
<script>
    function openCalendar(obj){
        var cal = IBSheet.showCalendar(
            {
                Data:obj.value,
                Buttons:7,
            },{
                Mouse: true
            },
            function(dd){
                //dd는 timestamp 값이므로 yyyy-MM-dd 형식으로 변환
                obj.value = IBSheet.dateToString(dd, 'yyyy-MM-dd');
            }
        );
        // 달력 팝업을 임시로 저장
        IBSheet.tempCal = cal;
    }
    //달력 닫기
    document.body.addEventListener("click",function(){
        if(IBSheet.tempCal){
            IBSheet.tempCal.close();
            IBSheet.tempCal = null;
        }
    });

</script>
<input type="text" onfocus='openCalendar(this)' readOnly>
```

## 그외 참고 기능
### 1) document 내에 시트 객체 접근하기

document 내에 모든 시트 객체는  IBSheet객체에 등록됩니다.<br>
따라서 IBSheet[0].getValue(row, col) 식으로 시트에 접근 할 수 있습니다.<br>
*주의: sheet.dispose()함수를 호출시 IBSheet에 등록되었던 객체는 null 로 변경됩니다.<br>이 경우 시트는 제거되었으나 IBSheet.length를 통해 시트의 개수에는 변화가 없습니다.*

### 2) 모든 시트 객체 제거하기

IBSheet[0].[disposeAll](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/static/dispose-all)함수를 통해 document에 생성된 모든 시트객체를 제거할 수 있습니다.

### 3) 모든 시트 객체가 생성된 후 발생하는 이벤트

document 내에 모든 시트객체가 생성된 후 수행해야 하는 로직이 있다면, IBSheet.[onRenderFirstFinishAll](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/static/on-render-first-finish-all) 이벤트에 정의 하시면 됩니다.<br>