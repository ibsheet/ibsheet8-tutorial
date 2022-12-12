# 7. 파일저장/업로드 (import/export)
## client 기반의 import/export 방법
javascript를 이용하여 서버의 도움없이 엑셀이나 txt 파일을 다운로드 하거나 업로드 할 수 있습니다.<br/>
그리드 내용과 더불어 타이틀이나 조회조건등 부수적인 내용을 함께 다운로드 하실 수 있습니다.<br/>
해당 기능을 사용하기 위해서는 [JSZip](https://stuk.github.io/jszip/) 라이브러리가 필요합니다. (/plugins 폴더에 배포)

### 1) 참고 사항

- 사용자 브라우저가 IE10 이상 되어야 하며, 실제 사용은 Edge 나 Chrome  이상의 브라우저를 권고합니다.
- 엑셀 파일에 대한 다운로드,업로드는 xlsx만 가능하며, xls 파일은 지원하지 않습니다.
- DRM(문서암호화)가 설정된 파일에 대해서는 업로드가 불가합니다.
- MS오피스가 아닌 외부 프로그램을 통해 만들어진 xlsx 파일은 업로드가 불가할 수 있습니다.

### 2) 파일 저장 함수
[exportData](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/core/export-data)함수를 통해 시트의 내용을 다운로드 합니다.
```js
function download(){
    mySheet.exportData({
        fileName:"시트 데이터.xlsx", //뒤에 확장자에 따라 xlsx,txt,csv 등으로 다운로드 됩니다.
        sheetName:"지역별 인구 분포"  //엑셀파일로 다운로드시 worksheet 이름
    });
}
```

### 3) 파일 업로드 함수
[importData](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/core/import-data)함수를 통해 엑셀이나 텍스트 파일의 내용을 시트 위로 업로드 할 수 있습니다.
```js
function upload(){
    mySheet.importData({
        startRow:4 // 몇 번째 행부터 읽어들인지 여부
    });
}
```
---
## server 기반의 import/export 방법
시트의 조회된 데이터를 서버로 전송하여 엑셀이나 txt파일로 다운로드 하거나, 사용자 PC의 파일을 서버로 전송하여 서버에서 파싱된 데이터를 시트에 로드할 수 있습니다.<br>
그리드 내용과 더불어 타이틀이나 조회조건등 부수적인 내용을 함께 다운로드 하실 수 있습니다.<br/>
해당 기능을 사용하기 위해서는 ibsheet-excel.js 파일이 필요합니다. (/plugins 폴더에 배포)

### 1) 참고 사항
- java기반 서버 사용시 APACHE POI 3.13 이상의 라이브러리를 사용합니다. (JDK 1.5 이상)
- 사용자 브라우저 버젼에 무관하게 다운로드/업로드 할 수 있으나, 사용자 수나 데이터 양에 따라 서버에 부하가 걸리 수 있습니다.
- DRM(문서암호화)가 설정된 파일에 대해서 서버에서 복호화 후 로드할 수 있습니다.(서버에 복호화 모듈 필요)
- MS오피스가 아닌 외부 프로그램을 통해 만들어진 xlsx 파일은 업로드가 불가할 수 있습니다.

***다음과 같은 파일이 제품과 함께 배포 됩니다.***<br>
<div style='border:1px solid orange'>
<p style='font-weight:700'>JAVA 기반 라이브러리</p>
ibsheet8-1.0.x.jar<br/>
commons-codec-1.6.jar<br/>
commons-logging-1.1.3.jar<br/>
batik-all-xml.jar<br/>
poi-3.13.jar<br/>
poi-ooxml-3.13.jar<br/>
poi-ooxml-schemas-3.13.jar<br/>
xmlbeans-2.3.0.jar<br/>
<br/>
<p style='font-weight:700'>JSP 파일</p>
DirectDown2Excel.jsp<br/>
DirectLoadExcel.jsp<br/>
Down2Excel.jsp<br/>
Down2Text.jsp<br/>
LoadExcel.jsp<br/>
LoadText.jsp<br/>
</div>
<span style='font-size:11px'>IIS 서버 사용시 관련 dll이 배포 됩니다.</span>



### 2) 파일 저장 함수
시트 생성시 (Cfg)Export 속성을 통해 서버측 URL을 정의 합니다.
```js
var options = {
    Cfg: {
        Export: {
            Down2ExcelUrl: "/sheet/jsp/Down2Excel.jsp",
            Down2TextlUrl: "/sheet/jsp/Down2Text.jsp"
        }
    }
}
```
[down2Text](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/excel/down-to-text), [down2Excel](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/excel/down-to-excel) 함수를 이용하여 다운로드 합니다.
```js
mySheet.down2Excel({
    fileName:"금년도 결산.xlsx",
    excelFontSize: 19
})
```

### 3) 파일 업로드 함수
시트 생성시 (Cfg)Export 속성을 통해 서버측 URL을 정의 합니다.
```js
var options = {
    Cfg: {
        Export: {
            LoadExcelUrl: "/sheet/jsp/LoadExcel.jsp",
            LoadTextUrl: "/sheet/jsp/LoadText.jsp"
        }
    }
}
```
loadExcel, loadText 함수를 이용하여 업로드 합니다.
```js
mySheet.loadExcel({mode: "NoHeader});
```