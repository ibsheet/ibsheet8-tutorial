# 2. 객체 생성과 초기화
## 파일구성

### 필수 파일
- ibsheet.js (제품의 코어 파일)
- ibleaders.js (라이선스 파일)
- /locale/ko.js (메세지 파일)
- /css/default/main.css (디자인 테마 파일)

### 선택 파일
- /plugins/ibsheet-common.js (공통속성 설정 파일)
- /plugins/ibsheet-dialog.js (다이얼로그 관련 기능 파일)
- /plugins/ibsheet-excel.js (파일 다운로드/업로드 관련 기능 파일)

### 기타
- /css/compatible/(dark or light)/main.css (css3 미지원 브라우져용 테마 파일)

<br>
<br>


## 객체 생성
1) 파일 import
```html
<!----- ibsheet 기본 모듈 ----->
<link rel="stylesheet" type="text/css" href="ibsheet/css/default/main.css">
<script src="ibsheet/ibsheet.js"></script>
<script src="ibsheet/ibleaders.js"></script>
<script src="ibsheet/locale/ko.js"></script>


<!----- ibsheet 선택 모듈 ----->
<script src="ibsheet/plugins/ibsheet-common.js"></script>
<script src="ibsheet/plugins/ibsheet-dialog.js"></script>
<script src="ibsheet/plugins/ibsheet-excel.js"></script>
```

2) 간단한 객체 생성

- 시트를 생성할 div객체를 추가하고 div객체에 너비,높이를 설정합니다.
- IBSheet.create() 함수를 통해 시트객체를 생성합니다.
```html
<div id="sheetEl" style="width:100%;height:500px"/>
<script>
IBSheet.create({
  id: "mySheet",
  el: "sheetEl",
  options: {
    Cols:[
      {Header:"사원명", Type:"Text", Name: "sanm"},
      {Header:"사원번호", Type:"Text", Name: "said"},
      {Header:"입사일", Type:"Text", Name: "edate"},
      {Header:"부서", Type:"Text", Name: "dept"}
    ]
  }
});
</script>
```

![생성된시트](firstSheet.png)

3) 객체 생성 기본 속성

```javascript
var OPT = {
  Cfg: {}, //시트 전체적인 기능 설정
  Cols: [{},{}...], //각 컬럼의 기능 설정
  Events: {} //이벤트 설정
};
IBSheet.create({
  id: "sheet1",
  el: "sheetDIV",
  options: OPT
})
```
- Cfg 속성

페이징 조회 여부나 전체 데이터의 편집가능/불가능 여부와 같은 시트 전체적인 기능을 설정합니다.

- Cols 속성

각 컬럼의 타이틀, 이름, 타입, 너비 등을 설정합니다.

- Events 속성

시트에서 일어나는 액션에 대한 이벤트를 설정합니다.

<br>
<br>
<br>

### [**options**](https://docs.ibleaders.com/ibsheet/v8/manual/#docs/start/init-structure) 속성의 구성
