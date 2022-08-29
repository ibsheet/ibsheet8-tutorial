# 3. 함수와 이벤트

## 함수
### 함수 사용법

<a href='https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/method' target="_blank"> 함수 사용법 기초 </a>


### 자주 사용되는 함수
|함수명|함수 설명|
|---|---|
|getValue|셀의 값을 얻음|
|setValue|셀의 값을 수정|
|getString|포멧이 적용된 셀의 값을 얻음|
|getAttribute|행,열,셀의 속성을 얻음|
|setAttribute|행,열,셀의 속성을 수정|
|addRow|행 추가|
|removeRow|행 삭제|
|deleteRow|행의 상태flag를 삭제로 변경|
|removeAll|전체 데이터 제거|
|dispose|시트객체 초기화|
|focus|특정 위치로 포커스 이동|
|loadSearchData|데이터 로드|
|getSaveJson(getSaveString)|시트 데이터 추출|


```javascript
//포커스된 셀의 값 얻기
mySheet.getValue( mySheet.getFocusedRow(), mySheet.getFocusedCol());
//속성 값 설정
mySheet.setAttribute( mySheet.getFirstRow(), getFocusedCol(), "CanEdit", 0);
// 행추가 
mySheet.addRow({next: mySheet.getFocusedRow() ,init:{movieNm:"탑건 매버릭",movieCd:"1234567" }});
// 행 삭제
mySheet.removeRow( mySheet.getFocusedRow() );
```

### (참고) Row, Col 객체 Selector 함수
|함수명|함수 설명|
|---|---|
|getFocusedRow|포커스가 있는 행 객체 얻음|
|getFirstRow|데이터 행 중 첫번째 행 객체 얻음|
|getFirstVisibleRow|데이터 행 중 첫번째 보여지는 행 객체 얻음|
|getLastRow|데이터 행 중 마지막 행 객체 얻음|
|getLastVisibleRow|데이터 행 중 마지막 보여지는 행 객체 얻음|
|getRowById|id를 통한 행 객체 얻음|
|getRowByIndex|index(데이터행 기준)를 통한 행객체 얻음|
|getPrevRow|특정 행의 이전행을 얻음|
|getNextRow|특정 행의 다음행을 얻음|
|getDataRows|전체 데이터객체 얻음|
|getShownRows|현재 화면에 보여지는 행을 모두 얻음|
|getChildRows|특정 행의 자식행(트리구조)을 모두 얻음|
|getRowsByChecked|특정 열(Bool타입)을 기준으로 체크된 행을 모두 얻음|
|getFocusedCol|포커스가 있는 열 이름 얻음|
|getFirstCol|첫번째 열의 이름 얻음|
|getLastCol|마지막 열의 이름 얻음|
|getPrevCol|특정 열을 기준으로 이전 열을 얻음|
|getNextCol|특정 열을 기준으로 다음 열을 얻음|
|getCols|전체 열 이름 얻음|

## 이벤트
### 이벤트 정의

1. 시트 객체 생성시 이벤트 정의 (추천)
```javascript
IBSheet.create({
    id: "mySheet",
    el: "SHEET_DIV",
    options: {
        Cfg: {},
        Cols: [
             LeftCols:[
                {Header:"순번", Type:"Int", Name:"SEQ", Align:"center"},
                {Header:"확정완료", Type:"Bool", Name:"confirmed"}
            ],
            Cols:[
                {Header:"영화명", Type:"Text", Name:"movieNm"},
                {Header:"영화코드", Type:"Text", Name:"movieCd"},
                {Header:"제작국가", Type:"Text", Name:"nationAlt"},
                {Header:"장르", Type:"Text", Name:"genreAlt"},
                {Header:"개봉일", Type:"Date", Name:"openDt", DataFormat:"yyyyMMdd", Format: "yyyy-MM-dd" },
                {Header:"영화정보확인", Type:"Button", Name:"movie_info_btn", ButtonText:"확인"}
            ],
        ],
        Events: {
            onClick: function(evtParam){
                if(evtParam.col === "movie_info_btn"){
                    const openDt = evtParam.sheet.getString( evtParam.row, "openDt" );
                    alert(`${openDt}에 개봉된 영화입니다.`);
                }
            }
        }
    }
})
```

2. 생성된 시트에 이벤트 bind (비추천)
```javascript
mySheet.bind("onBeforeChange",function(evtParam){
    var sheet = evtParam.sheet;
    if(sheet.getValue(evtParam.row,"confirmed") === 1){
        alert("이미 확정된 데이터는 수정하실 수 없습니다.");
        return evtParam.oldval;
    } 
});
```
### 이벤트 특징
1. 이벤트의 파라미터
    모든 이벤트의 파라미터는 1개(object형)이며, 이벤트에 따라 파라미터에 각기 다른 속성 값을 갖습니다.
2. 이벤트의 리턴값
    일부 이벤트는 리턴값을 통해 다음 이벤트의 실행을 중단하거나, 사용자 입력을 무효화 할 수 있습니다.

### 자주 사용되는 이벤트
|이벤트명|이벤트 설명|
|---|---|
|onRenderFirstFinish|시트 객체 생성시 1회 호출되는 이벤트(<mark>시트 생성은 비동기임</mark>)|
|onAfterClick|클릭 이벤트(onClick에 비해 나중에 호출됨)|
|onBeforeChange|값 수정시 발생하는 이벤트|
|onSearchCallback|데이터 로드 전 발생하는 이벤트|
|onSearchFinish|데이터 로드 완료 후 발생하는 이벤트|
|onExportFinish|excel이나 text 형식 파일 다운로드 완료 후 발생 이벤트|


**조회 완료 후 데이터 수정 (loop문과 렌더링)**
```javascript
{
    Events:{
        onSearchFinish: function(ep){
            //전체 데이터 행객체
            const rows = ep.sheet.getDataRows();
            //제작국가가 미국인 영화에 대해 배경색으로 노란색으로 설정

            // 올바르지 않은 예
            for(let i = 0 ; i < rows.length ; i++){
                if(ep.sheet.getValue(rows[i], "nationAlt") === "미국"){
                    ep.sheet.setAttribute(rows[i], null, "Color", "#FFFF00");
                }
            }

            // 올바른 예
            for(let i = 0 ; i < rows.length ; i++){
                if(ep.sheet.getValue(rows[i], "nationAlt") === "미국"){
                    ep.sheet.setAttribute(rows[i], null, "Color", "#FFFF00", 0);
                }
            }
            // 변경된 데이터를 화면에 표시
            ep.sheet.renderBody();
        }
    }
}
```

### 주요 이벤트 발생 순서
1. 조회 관련 이벤트
doSearch나 loadSearchData()를 통해 시트에 데이터가 로드될 때, 다음 순서대로 이벤트가 발생합니다.<br>
***onSearchCallback -> onBeforeDataLoad -> onDataLoad -> onSearchFinish***

2. 저장 관련 이벤트
doSave()함수 호출시 다음 순서대로 이벤트가 발생합니다.<br>
***onSave -> onBeforeSave -> 시트 데이터 서버 전송 -> onAfterSave***

3. 셀편집 이벤트
사용자가 데이터 편집시 다음 순서대로 이벤트가 발생합니다.<br>
***onStartEdit -> onShowEdit -> onEndEdit -> onAfterEdit -> onBeforeChange -> onAfterChange***



