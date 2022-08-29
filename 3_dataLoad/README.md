# 3. 데이터 로드
## 1) 객체 생성 시점 데이터 로드

시트객체 생성시 options 속성에 "data" 속성을 통해 데이터를 로드할 수 있습니다.

```javascript
IBSheet.create({
  id: "mySheet",
  el: "sheetEl",
  options: {
    Cols:[
      {Header:"사원명", Type:"Text", Name:"sanm"},
      {Header:"사원번호", Type:"Text", Name:"said"},
      {Header:"입사일", Type:"Date", Name:"edate", DataFormat:"yyyyMMdd", Format: "yyyy-MM-dd" },
      {Header:"부서", Type:"Enum", Name:"dept", Enum:"|총무|인사|기획|영업1|영업2|기술1", EnumKeys:"|A01|A02|A03|B01|B02|T01" }
    ]
  },
  // data 속성 사용시 시트 생성과 동시에 데이터가 로드됨
  "data": [
    {"sanm":"홍길동", "said":"P2008031", "edate": "20080403", "dept": "A01"},
    {"sanm":"김대한", "said":"P2015118", "edate": "20150811", "dept": "T01"},
    {"sanm":"이경웅", "said":"P2004007", "edate": "20040216", "dept": "B02"},
  ]
});
```
## 2) <a href="https://docs.ibleaders.com/ibsheet/v8/manual/#docs/appx/data-structure"  target='_blank'>조회 데이터 구조</a>
```javascript
{
    "total":5140,  //Paging 조회시에만 필요(최초 조회때 한번만 필요)
    "data":[
        {column1Name:"value11", column2Name:"value21".....},
        {column1Name:"value12", column2Name:"value22".....},
        ...
    ]
}
```

## 3) 데이터 로드 함수
### i) ajax를 통한 데이터 로드 (<a href='https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/do-search' target='_blank'>doSearch</a>)
ajax를 통해 특정 URL로부터 json데이터를 받아, 로딩합니다.

[조회 데이터 파일 작성 data.json]
```javascript
{
    "data": [
        {"sanm":"김춘수", "said":"P2019008", "edate": "20190203", "dept": "B01"},
        {"sanm":"안수정", "said":"P2007049", "edate": "20070503", "dept": "A03"},
        {"sanm":"한민국", "said":"P2009088", "edate": "20090406", "dept": "B01"},
        {"sanm":"민기영", "said":"P2009029", "edate": "20070314", "dept": "B01"},
      ]
}
```

```html
<button type="button" onclick="retrieve()">조회</button>
<script>
// 데이터 조회
function retrieve(){
  const url = "./data.json";
  const param = {
      url: url,
      param: "curPage=1",
      method: "GET",
  }
  mySheet.doSearch(param);
}
</script>
``` 
### ii) 이미 갖고있는 데이터를 로드하기 (<a href='https://docs.ibleaders.com/ibsheet/v8/manual/#docs/funcs/load-search-data' target='_blank'>loadSearchData</a>)
window 안의 임의의 데이터를 로드하거나, 다른 라이브러리(axios,jquery등)로부터 얻은 데이터를 로드합니다.

#### axios cdn import
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.27.2/axios.min.js" integrity="sha512-odNmoc1XJy5x1TMVMdC7EMs3IVdItLPlCeL5vSUPN2llYKMJ2eByTTAIiiuqLg+GdNr9hF6z81p27DArRFKT7A==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

#### ibsheet  초기화
```html
<div id="sheetEl" style="width:100%;height:500px"/>
<script>
IBSheet.create({
  id: "mySheet",
  el: "sheetEl",
  options: {
    Cfg: {
        SearchMode: 0,
    },
    LeftCols:[
        {Header:"순번", Type:"Int", Name:"SEQ", Align:"center"}
    ],
    Cols:[
        {Header:"영화명", Type:"Text", Name:"movieNm"},
        {Header:"영화코드", Type:"Text", Name:"movieCd"},
        {Header:"제작국가", Type:"Text", Name:"nationAlt"},
        {Header:"장르", Type:"Text", Name:"genreAlt"},
        {Header:"개봉일", Type:"Date", Name:"openDt", DataFormat:"yyyyMMdd", Format: "yyyy-MM-dd" },
    ],
  }
});
</script>
```


<b><a href="https://www.kobis.or.kr/kobisopenapi/homepg/apiservice/searchServiceInfo.do" target="_blank">영화진흥원 영화목록 openapi</a></b>


#### 데이터 로드
```javascript
// 데이터 조회
function retrieve(){
    //영화진흥원 영화목록 openapi  (https://www.kobis.or.kr/kobisopenapi/homepg/apiservice/searchServiceInfo.do)
    const url = "https://kobis.or.kr/kobisopenapi/webservice/rest/movie/searchMovieList.json?key=f5eef3421c602c6cb7ea224104795888&itemPerPage=100&prdtEndYear=2000";
    axios({
        url: url,
        method: 'get',
        responseType: 'json'
    })
    .then(function (response) {
        //리턴데이터 확인
        // console.log(response); 
        
        // 데이터 가공
        const data = {"data":response.data.movieListResult.movieList};
        mySheet.loadSearchData(data); //비동기 형식 주의
    }).catch(function (error) {
        console.log("데이터 조회 중 오류가 발생하였습니다.");
    });
}
```
