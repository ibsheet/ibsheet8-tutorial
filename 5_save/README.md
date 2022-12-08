# 5. 데이터 수정과 저장

## 데이터 수정

### 1) 행의 상태(flag)

CanEdit 속성을 통해 시트 내에 행,열,셀에 대한 편집 가능 여부를 변경할 수 있습니다.<br>
조회 된 상태에서 값이 수정되면 각 행에는 **Changed**라는 flag값이 추가되고 배경색이 변경 됩니다.<br>
이 외에도 addRow()함수를 통해 추가되거나, deleteRow()를 통해 삭제 예정인 행의 경우 **Added** 나 **Deleted**라는 flag가 추가됩니다.<br>
```javascript
// 10번째 행객체 얻기
var row = sheet.getRowByIndex(10);
if(row["Changed"]){
	//해당 행은 수정된 내용이 있음.
}
if(row["Added"]){
	//신규로 추가된 행임.
}
if(row["Deleted"]){
	//삭제 예정행임
}
```


### 2) 데이터 수정 Validation 속성
EditMask 속성을 통해 열에 필요한 글자만 입력 가능하게 설정할 수 있습니다.

ResultMask 속성을 통해 편집 완료시점에서 입력한 값을 정규식을 통해 검사할 수 있습니다.

```js
var OPT = {

    "Cols":[
                                                        
        {
            Header:"계좌 번호", 
            Name:"accountNo", 
            Type:"Text", 
            EditMask:"^[\\w\\-]*$"  //편집 중 영문,숫자와 "-" 만 입력 가능
        }, 
                                                       
        {
            Header:"전화 번호", 
            Name:"telNo", 
            Type:"Text", 
            ResultMask:"^0\\d{1,2}-?\\d{3,4}-?\\d{4}$", // 편집 완료 후 검사 검사
            ResultMessage:"전화번호 형식으로 입력해 주세요." // 잘못된 입력시 보여질 경고 메세지 
        },
        ...
    ]
}
```
### 3) 데이터 수정시 발생 이벤트
텍스트 필드를 편집시 다음과 같은 이벤트가 순차적으로 발생합니다.
```
편집 시작 -> onStartEdit -> onShowEdit -> onEndEdit -> onAfterEdit
```

사용자가 데이터를 수정 시(수정 완료시), 다음과 같은 이벤트가 발생합니다.
```
값 변경 후 -> onBeforeChange -> Col.OnChange,Col.OnSame -> onAfterChange
```
<hr/>

## 데이터 저장
### 1) 데이터 수정여부 확인
수정된 데이터의 유무는 hasChangedData() 함수를 통해 확인하실 수 있습니다.
```js
function save(){
    // 수정된 데이터가 있는지 판별
    if(mySheet.hasChangedData()) {
        // ... 저장 로직 ...
    }else{
        alert("수정 된 데이터가 없습니다.");
    }
}

```


### 2) 데이터 추출
수정된 데이터는 getSaveJson()이나 getSaveString() 함수를 통해 추출할 수 있습니다.
```js
var json = mySheet.getSaveJson(); // 수정된 내용을 JSON object형식으로 추출
var queryStr = mySheet.getSaveString(); // 수정된 내용을 querystring 형식으로 추출
```
추출된 내용에는 데이터 외에 행의 상태(flag)가 STATUS라는 이름으로 담겨 있습니다.
```
// json 내용
{"data":[
    {"id":"AR4","SEQ":4,"sCheck":"0","sNation":"일본","sTitle":"레전드 오브 타잔","sShare":10.6,"sCount":692133,"sDate":"20160629","STATUS":"Changed"},
    {"id":"AR51","Def":"R","Parent":"","Next":"AR5","Prev":"AR4","SEQ":5,"sCheck":"0","sNation":"한국","sTitle":"외계인","sShare":11,"sCount":123456,"sDate":"20220816","STATUS":"Added"},
    {"id":"AR9","SEQ":10,"sCheck":"0","sNation":"미국","sTitle":"컨저링 2","sShare":2.7,"sCount":184519,"sDate":"20160609","STATUS":"Deleted"}
]}

// queryStr 내용
id=AR4&SEQ=4&sCheck=0&sNation=%EC%9D%BC%EB%B3%B8&sTitle=%EB%A0%88%EC%A0%84%EB%93%9C%20%EC%98%A4%EB%B8%8C%20%ED%83%80%EC%9E%94&sShare=10.6&sCount=692133&sDate=20160629&STATUS=Changed&id=AR51&Def=R&Parent=&Next=AR5&Prev=AR4&SEQ=5&sCheck=0&sNation=%ED%95%9C%EA%B5%AD&sTitle=%EC%99%B8%EA%B3%84%EC%9D%B8&sShare=11&sCount=123456&sDate=20220816&STATUS=Added&id=AR9&SEQ=10&sCheck=0&sNation=%EB%AF%B8%EA%B5%AD&sTitle=%EC%BB%A8%EC%A0%80%EB%A7%81%202&sShare=2.7&sCount=184519&sDate=20160609&STATUS=Deleted
```

추출과정에서 필수값에 대한 오류가 발생시 다음과 같이 추출됩니다.
```
// json 내용
{"data":[],"Message":"RequiredError","Code":"IBS010","col":"sTitle"}

// queryStr 내용
RequiredError|IBS010|AR51|sTitle
```
```js
var saveData = mySheet.getSaveString();
// 필수값 확인
if(saveData.indexOf("RequiredError")==-1){
    // ... 저장 로직 ...
}else{
    var errCode = saveData.split("|");
    var hRow = mySheet.getHeaderRows();
    var colTitle = mySheet.getString(hRow[hRow.length - 1] , errCode[3]);
    alert(`${colTitle}열은 필수 입력컬럼 입니다.`);
    mySheet.focus(mySheet.getRowById(errCode[2]),errCode[3] );
    return;
}
```


### 3) 수정데이터 전송

전송 전에 getRowsByStatus()함수를 통해 서버로 전송될 데이터 건수를 확인하게 할 수 있습니다.
```js
// 신규행
var addRowCnt = mySheet.getRowsByStatus("Added,!Deleted").length;
// 수정행
var chgRowCnt = mySheet.getRowsByStatus("Changed,!Added,!Deleted").length;
// 삭제행
var delRowCnt = mySheet.getRowsByStatus("Deleted").length;
if(confirm(`신규 : ${addRowCnt}건\n수정 : ${chgRowCnt}건\n삭제 : ${delRowCnt}건\n을 저장하시겠습니까?`)){
    // ... 저장 로직 ...
}
```


시트가 제공하는 ajax함수나 외부라이브러리(jquery, axios)를 통해 서버로 수정한 데이터를 전송하고, 이에 대한 결과를 받습니다.
```js
    // 시트의 ajax 사용 예(axios나 jquery를 사용하셔도 됩니다.)
    mySheet.ajax({
        url : "./save.jsp",
        param : saveData,
        method : "post",
        callback: function(res, data, resXml, response){
            // ... 저장 완료 처리 ...
        }
    });
```

### 4) 저장 완료 처리(상태 데이터 클리어)
정상적으로 저장이 완료된 후에는 acceptChangedData()함수를 통해 행의 상태를 클리어 할 수 있습니다.

acceptChangedData()함수는 기존 행의 상태에 따라 다음과 같이 동작합니다.
1) 행의 상태가 Added나 Changed 인 경우 - 행의 flag를 제거하여 조회 상태로 변경
2) 행의 상태가 Deleted 인 경우 - 행을 제거

```js
mySheet.ajax({
    ...,
    callback: function(res, data, resXml, response){
        // 저장 완료 처리
        if( data && JSON.parse(data)?.IO?.Result > -1){
            mySheet.acceptChangedData();
        }
    }
})
```

**완성된 저장 함수**
```js
function save(){
    // 수정된 데이터가 있는지 판별
    if(mySheet.hasChangedData()) {
        var saveData = mySheet.getSaveString();
        // 필수값 확인
        if(saveData.indexOf("RequiredError")==-1){
            var addRowCnt = mySheet.getRowsByStatus("Added,!Deleted").length;
            // 수정행
            var chgRowCnt = mySheet.getRowsByStatus("Changed,!Added,!Deleted").length;
            // 삭제행
            var delRowCnt = mySheet.getRowsByStatus("Deleted").length;
            if(confirm(`신규 : ${addRowCnt}건\n수정 : ${chgRowCnt}건\n삭제 : ${delRowCnt}건\n을 저장하시겠습니까?`)){
                mySheet.ajax({
                    url : "./save.jsp",
                    param : saveData,
                    method : "post",
                    callback: function(res, data, resXml, response){
                        // 저장 완료 처리
                        if( data && JSON.parse(data)?.IO?.Result > -1){
                            mySheet.acceptChangedData();
                        }
                    }
                });
            }
        }else{
            var errCode = saveData.split("|");
            var hRow = mySheet.getHeaderRows();
            var colTitle = mySheet.getString(hRow[hRow.length - 1] , errCode[3]);
            alert(`${colTitle}열은 필수 입력컬럼 입니다.`);
            mySheet.focus(mySheet.getRowById(errCode[2]),errCode[3] );
            return;
        }
    }else{
        alert("수정 된 데이터가 없습니다.");
    }
}
```


### 데이터 저장 함수
위 과정을 한번에 수행하는 doSave()함수를 사용하실 수 있습니다.
이 함수에서는 데이터 추출, 전송, 완료처리가 순차적으로 일어나게 됩니다.
```js
// 해당 시트에 수정된 데이터를 추출하여 지정한 URL로 전송합니다.
// 리턴되는 결과 json에 따라 상태를 클리어합니다.
mySheet.doSave({
    url: "./save.jsp",
    saveMode: 2, //수정된 데이터만 전송
    quest: 1 //저장 전 confirm  표시
});
```

### 저장 JSON  구조
```json
{
    "IO":{
        "Result":0  //Result값이 양수( 0 포함 )이면 저장성공, 음수이면 저장실패로 판단합니다.
    }
}
```



