  
![header](https://capsule-render.vercel.app/api?type=soft&fontColor=ffffff&height=200&section=header&text=DW%20Academy&fontSize=50)


## 🎈 개요
<br>
영화 예매 플랫폼 <b>DW Academy</b>

link : <https://github.com/jjgod66/dw-final-group1>
<br><br><br>

## 📚 개발 환경
* <b>Language</b> : JAVA, Javascript, HTML5, CSS3
* <b>Framework/Flatform</b> : Spring, eGovFrame, myBatis, Bootstrap, jQuery
* <b>Server</b> : Apache Tomcat 8.5
* <b>DB</b> : Oracle 11g
* <b>Tool</b> : Eclipse, GitHub
<br><br><br>
## 🖥 담당 역할
#### 플랫폼 내 주관리자, 지점관리자 이용 페이지 구현
<br><br>

## ⌨ 상세 설명

### 주관리자 - 메인
  
![sys_main01](https://github.com/jjgod66/jjgod66/assets/125471355/8de7ec21-9c65-4c93-b9e0-e34551557427)
<br><br>

### 주관리자 영화관리 - 등록 영화 목록
  
![sys_mov01](https://github.com/jjgod66/jjgod66/assets/125471355/664a5df6-1071-4c7d-bcd7-f3affe52068f)
<br><br>

### 주관리자 영화관리 - 등록 영화 수정
  
![sys_mov02](https://github.com/jjgod66/jjgod66/assets/125471355/02d559ef-04c4-40b7-9761-ab74ecd7b99b)
![sys_mov03](https://github.com/jjgod66/jjgod66/assets/125471355/69f16f88-a330-4129-b1b9-8028766875d0)
<br><br>

영화 예매 플랫폼 사이트이다보니 사이트 전반적으로 사진이나 동영상을 다뤄야 하는 경우가 많았습니다.

그래서 따로 공통영역 부분으로 빼내어 메서드들을 만들었습니다.
```java
// 공통영역 부분

// 사진파일 저장
	private String savePicture(MultipartFile multi, String oldPicture, String item_cd, String type) throws Exception {
		
		String fileName = null;
		
		// 파일 유무 확인
		if (!(multi == null || multi.isEmpty() || multi.getSize() > 1024 * 1024 * 10)) {
			// 파일 저장 폴더 설정
			String imgPath = "";
			if (type.equals("moviePoster")) {
				imgPath = this.moviePicUploadPath + File.separator + item_cd + File.separator + "mainPoster";
			} else if (type.equals("movieImg")) {
				imgPath = this.moviePicUploadPath + File.separator + item_cd + File.separator + "pictures";
			} else if (type.equals("movieVideo")) {
				imgPath = this.moviePicUploadPath + File.separator + item_cd + File.separator + "videos";
			} else if (type.equals("productImg")) {
				imgPath = this.storePicUploadPath + File.separator + item_cd;
			}
			fileName = multi.getOriginalFilename();
			File storeFile = new File(imgPath, fileName);
			
			storeFile.mkdirs();
			
			// local HDD에 저장
			multi.transferTo(storeFile);
			
			if (!oldPicture.isEmpty()) {
				File oldFile = new File(imgPath, oldPicture);
				if (oldFile.exists()) {
					oldFile.delete();
				}
			}
		}
		
		return fileName;
	}

// 사진 읽기
@RequestMapping("/common/getPicture")
public ResponseEntity<byte[]> getPicture(String name, String item_cd, String type) throws IOException  {
  
  InputStream in = null;
  ResponseEntity<byte[]> entity = null;
  String imgPath = "";
  if (type != "" && type != null) {
    if (type.equals("moviePoster")) {
      imgPath = this.moviePicUploadPath + File.separator + item_cd + File.separator + "mainPoster";
    } else if (type.equals("movieImg")) {
      imgPath = this.moviePicUploadPath + File.separator + item_cd + File.separator + "pictures";
    } else if (type.equals("productImg")) {
      imgPath = this.storePicUploadPath + File.separator + item_cd;
    } else if (type.equals("eventThumb")) {
      imgPath = this.eventPicUploadPath + File.separator + item_cd + File.separator + "thumb";
    } else if (type.equals("eventImg")) {
      imgPath = this.eventPicUploadPath + File.separator + item_cd + File.separator + "img";
    } else if (type.equals("memberPic")) {
      imgPath = this.memberPicUploadPath + File.separator + item_cd;
    } else if (type.equals("back")) {
      imgPath = this.photoTicketUploadPath + File.separator + "back";
    } else if (type.equals("front")) {
      imgPath = this.photoTicketUploadPath + File.separator + "front";
    } else if (type.equals("qr")) {
      imgPath = this.qrUploadPath;
    }
  }
  
  try {
    in = new FileInputStream(new File(imgPath, name));
    entity = new ResponseEntity<byte[]>(IOUtils.toByteArray(in), HttpStatus.CREATED);
  } catch (FileNotFoundException e) {
    e.printStackTrace();
    entity = new ResponseEntity<byte[]>(HttpStatus.INTERNAL_SERVER_ERROR);
  } finally {
    in.close();
  }
  
  return entity;
}
// 동영상 읽기
@RequestMapping(value = "/common/getVideo")
   public ResponseEntity<ResourceRegion> getVideo(@RequestHeader HttpHeaders headers, String movie_cd, String movie_pre_path) throws IOException {
      logger.info("VideoController.getVideo");
      UrlResource video = new UrlResource("file:"+ this.moviePicUploadPath + File.separator + movie_cd + File.separator + "videos" + File.separator + movie_pre_path);
      ResourceRegion resourceRegion;

      final long chunkSize = 1000000L;
      long contentLength = video.contentLength();

      Optional<HttpRange> optional = headers.getRange().stream().findFirst();
      HttpRange httpRange;
      if (optional.isPresent()) {
          httpRange = optional.get();
          long start = httpRange.getRangeStart(contentLength);
          long end = httpRange.getRangeEnd(contentLength);
          long rangeLength = Long.min(chunkSize, end - start + 1);
          resourceRegion = new ResourceRegion(video, start, rangeLength);
      } else {
          long rangeLength = Long.min(chunkSize, contentLength);
          resourceRegion = new ResourceRegion(video, 0, rangeLength);
      }

      return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT)
              .contentType(MediaTypeFactory.getMediaType(video).orElse(MediaType.APPLICATION_OCTET_STREAM))
              .body(resourceRegion);
  }
```
### 지점관리자 상영영화관리 - 상영시간표

![thr_mov01](https://github.com/jjgod66/jjgod66/assets/125471355/6e506063-648d-4e5a-b20d-0597e159275b)

관리자가 본인이 선택하는 시간대에 새로운 상영영화를 등록할 수 있는지 확인할 수 있는 상영시간표를 구현해야 했습니다.

선택한 상영영화의 시작시간이 현재시간 기준보다 이후인지, 그리고 종료시간이 영화관 마감시간보다 이전인지 우선 체크를합니다.

괜찮다면 비동기로 해당 상영영화가 이미 등록된 다른 상영영화들과 겹치지 않는지 DB에서 확인합니다.
```javascript
// [JS] 서버로 가서 추가 가능한 시간인지 체크
function checkScreenTimeClash(house_no, startTime, endTime, screen_cd) {

  // 현재시간 이후 시간인지 먼저 체크.
  if (startTime < new Date()) {
    if ($('.addedBox').length > 0) {
      $('.addedBox').addClass('cantBeAdded');
    } else if ($('.modifyBox').length > 0) {
      $('.modifyBox').addClass('cantBeAdded');
    }
    return;
  }
  
  // 끝나는 시간이 마감시간(익일03시) 이전인지 체크.
  console.log('endtime', endTime.getHours());
  if (endTime.getHours() >= 3 && endTime.getHours() < 7 ) {
    if ($('.addedBox').length > 0) {
      $('.addedBox').addClass('cantBeAdded');
    } else if ($('.modifyBox').length > 0) {
      $('.modifyBox').addClass('cantBeAdded');
    }
    return;
  }
  
  // 이미 등록된 다른 상영영화와 시간표 충돌이 없는지 체크 (등록 가능한지)
  let data = {
      "house_no" : house_no,
      "startTime" : startTime,
      "endTime" : endTime,
      "screen_cd" : screen_cd
  }
  $.ajax({
    url : "<%=request.getContextPath()%>/thrAdmin/CheckScreenTimeClash",
    type: "post",
    data : JSON.stringify(data),
    contentType : "application/json",
    success : function(data) {
      if (data > 0) {
        console.log(data);
        if ($('.addedBox').length > 0) {
          $('.addedBox').addClass('cantBeAdded');
        } else if ($('.modifyBox').length > 0) {
          $('.modifyBox').addClass('cantBeAdded');
        }
      }
    },
    error : function(err) {
      console.log(err);
    }
  });
}
```
```java
@RequestMapping("/CheckScreenTimeClash")
public ResponseEntity<String> checkScreenTimeClash (@RequestBody Map<String, Object> data) throws ParseException, SQLException {
  ResponseEntity<String> entity = null;
  
  int house_no = Integer.parseInt((String)data.get("house_no"));

  // 상영영화 시작시간 구하기
      Instant instant = Instant.parse((String)data.get("startTime"));			// 문자열 파싱
      ZonedDateTime kstDateTime = instant.atZone(ZoneId.of("Asia/Seoul"));	// Instant를 ZonedDateTime으로 변환하여 한국 시간대로
      Date startTime = Date.from(kstDateTime.toInstant());

  // markTime : 영화상영시간이 익일0시 이후여도 해당 날짜가 기준날짜가 되게 세팅
  Date markTime = null;
  if (kstDateTime.getHour() <= 3) {
    markTime = Date.from(kstDateTime.minusDays(1).toInstant());
  } else {
    markTime = startTime;
  }
  
  // 상영영화 끝시간 구하기
  instant = Instant.parse((String)data.get("endTime"));
  kstDateTime = instant.atZone(ZoneId.of("Asia/Seoul"));
  Date endTime = Date.from(kstDateTime.toInstant());
  
  data.put("house_no", house_no);
  data.put("startTime", startTime);
  data.put("endTime", endTime);
  data.put("markTime", markTime);
  
  // 충돌하는 상영영화 갯수 (0이여야 등록가능)
  int clashCnt = thrAdminService.checkScreenTimeClash(data);
  
  try {
    entity = new ResponseEntity<String>("" + clashCnt, HttpStatus.OK);
  } catch (Exception e) {
    entity = new ResponseEntity<String>("FAIL", HttpStatus.INTERNAL_SERVER_ERROR);
  }
  return entity; 
}
```
![thr_mov02](https://github.com/jjgod66/jjgod66/assets/125471355/3be4fcd5-ad3e-44c4-aa8d-716542f9dffe)
<br><br>

### 관리자 영화관리 - 통계
  
* 일자별
  
![sys_mov04](https://github.com/jjgod66/jjgod66/assets/125471355/abf713c1-7ef9-474e-a4a7-ad051e87ea76)
```javascript
// [JS] Chart.js 사용
// 가져온 영화목록 chart를 위해 새 배열에 세팅
const movieList = new Array();
<c:forEach items="${movieList}" var="movie">
  <c:if test="${movie.SALES_DAY ne 0}">
    movieList.push({
      name : "${movie.MOVIE_NAME}",
      sales_day : "${movie.SALES_DAY}",
      sales_allday : "${movie.SALES_ALLDAY}"
    });
  </c:if>
</c:forEach>

// chart 랜덤 색깔 배정
var strRGBAList = new Array();
for (let i = 0; i < movieList.length; i++) {
  let RGB_1 = Math.floor(Math.random() * (255 + 1));
  let RGB_2 = Math.floor(Math.random() * (255 + 1));
  let RGB_3 = Math.floor(Math.random() * (255 + 1));
  let strRGBA = 'rgba(' + RGB_1 + ',' + RGB_2 + ',' + RGB_3 + ')';
  strRGBAList.push(strRGBA);
}

const movieNameList = movieList.map(function(e){
  return e.name
});
const movieSales_day_List = movieList.map(function(e){
  return e.sales_day
});
const movieSales_allday_List = movieList.map(function(e){
  return e.sales_allday
});

const ctx = $('.sales_day_chart');
const ctx2 = $('.sales_allday_chart');

const plugin = {
      id: 'customCanvasBackgroundColor',
      beforeDraw: (chart, args, options) => {
        const {ctx} = chart;
        ctx.save();
        ctx.globalCompositeOperation = 'destination-over';
        ctx.fillStyle = options.color || '#99ffff';
        ctx.fillRect(0, 0, chart.width, chart.height);
        ctx.restore();
      }
};

let chart_config_day = {
      type: 'pie',
      data:  {
          labels: movieNameList,
          datasets: [{
            label: 'My First Dataset',
            data: movieSales_day_List,
            hoverOffset: 4,
            backgroundColor: strRGBAList
          }],
        },
      options: {
      scales: {
        x: {
          display: false,
            grid: {
              display: false,
              drawTicks: false
            }
          },
            y: {
              display: false,
              beginAtZero: true,
              grid: {
                display: false,
                drawTicks: false
              }
            }
          },
          plugins: {
            title : {
              display: true,
              text: '매출액',
              font: {
                weight: 'bold',
                size: 18
              },
              padding: 20,
              align: 'center',
              position: 'bottom',
              fullSize: false
            },
            customCanvasBackgroundColor: {
              color: '#e9ecef',
            }
          }
      },
      plugins: [plugin],
}

let chart_config_allday = {
      type: 'pie',
      data:  {
          labels: movieNameList,
          datasets: [{
            label: 'My First Dataset',
            data: movieSales_allday_List,
            hoverOffset: 4,
            backgroundColor: strRGBAList
          }]
        },
      options: {
          scales: {
            x: {
            display: false,
              grid: {
                display: false,
                drawTicks: false
              }
            },
              y: {
                display: false,
                beginAtZero: true,
                grid: {
                  display: false,
                  drawTicks: false
                }
              }
          },
          plugins: {
            title : {
              display: true,
              text: '누적 매출액',
              font: {
                weight: 'bold',
                size: 18
              },
              padding: 20,
              align: 'center',
              position: 'bottom',
              fullSize: false
            },
            customCanvasBackgroundColor: {
              color: '#e9ecef',
            }
          }
      },
      plugins: [plugin],
}

new Chart(ctx, chart_config_day);
new Chart(ctx2, chart_config_allday);

// 테이블 정렬버튼 구현
var sortType = 'asc';
function sortTable(cellNum){
    sortType = (sortType === 'asc') ? 'desc':'asc';

    var table = document.getElementById('statisticsTable');
    var rows = table.rows;
    var chkSort = true;
   
    while (chkSort){
        chkSort = false;
        for (var i = 1; i < (rows.length - 1); i++) {
            var row = rows[i];
            if (cellNum == 0) {
	            var fCell = row.cells[cellNum].innerHTML.toLowerCase();
	            var sCell = row.nextElementSibling.cells[cellNum].innerHTML.toLowerCase();
            } else if (cellNum >= 1 ){
	            var fCell = Number(row.cells[cellNum].innerHTML.toLowerCase().replace(',', '').replace('%', ''));
	            var sCell = Number(row.nextElementSibling.cells[cellNum].innerHTML.toLowerCase().replace(',', '').replace('%', ''));
            } else {
	            var fCell = row.cells[cellNum].innerHTML.toLowerCase();
	            var sCell = row.nextElementSibling.cells[cellNum].innerHTML.toLowerCase();
            }
            if ((sortType === 'asc'  && fCell > sCell) || (sortType === 'desc' && fCell < sCell)) {
                row.parentNode.insertBefore(row.nextSibling, row);
                chkSort = true;
            }
        }
       }   
   }
```
* 영화별
  
![sys_mov05](https://github.com/jjgod66/jjgod66/assets/125471355/63f8593d-bbd7-4927-bd0f-2ae20ac26817)
<br><br>

### 관리자 이벤트 - 이벤트 등록

![event_regist](https://github.com/jjgod66/jjgod66/assets/125471355/9f5dd21a-7083-48a5-95d4-c5a4b10b8b26)

글작성 에디터로 썸머노트를 사용하였습니다. 이미지 첨부를 위하여 썸머노트에서 지원하는 드래그 앤 드랍 콜백함수를 사용하려 했습니다.

서버에서 이미지가 저장되어야 할 곳은 해당 이벤트 작성 글의 PK가 폴더명인 폴더에 저장되길 원했습니다.

그래서 이미지 저장을 3단계로 나눴습니다.

1. 글 작성 중 드래그앤드랍 즉시 서버 안의 Temp폴더 안에 저장

2. 글 등록완료 시에 Temp폴더의 해당 이미지 파일을 이벤트 작성글 폴더 안으로 이동

3. 스케줄러를 통하여 주기적으로 Temp폴더 내 파일 삭제

```javascript
function summernote () {
		$('.summernote').summernote({
			lang: 'ko-KR',
			height: 600,
			disableResizeEditor : true,
			fontNames: ['Arial', 'Arial Black', 'Comic Sans MS', 'Courier New','맑은 고딕','궁서','굴림체','굴림','돋움체','바탕체'],
			fontSizes: ['8','9','10','11','12','14','16','18','20','22','24','28','30','36','50','72'],
			toolbar: [
		// 		[groupName, [list of button]]
				['fontname', ['fontname']],
				['fontsize', ['fontsize']],
				['style', ['bold', 'italic', 'underline','strikethrough', 'clear']],
				['color', ['forecolor','color']],
				['table', ['table']],
				['para', ['ul', 'ol', 'paragraph']],
				['height', ['height']],
				],
			callbacks : {
				onImageUpload : function(files, editor, welEditable){
					if ($('.note-editable img').length > 0) {
						alert('이미지는 1개만 첨부 가능합니다.');
						return;
					}
					
					for (let i = files.length - 1 ; i > -1; i--) {
						if (files[i].size > 1024 * 1024 * 10) {
							alert("이미지는 10MB 미만이어야 합니다.");
							return;
						};
					};
					
					// 파일 서버로 보내기
					for (let i = files.length -1; i >= 0; i--) {
						console.log(this);
						sendFile(files[i], this);
					};
				},
				onMediaDelete : function(target){
					if(confirm("삭제하시겠습니까?")){
						deleteFile(target[0].src);
					}
				}
			}
		});
	}

// 이미지 temp폴더에 저장
function sendFile(file, el) {
		let form_data = new FormData();
		form_data.append("file", file);
		$.ajax({
			url : '<%=request.getContextPath() %>/common/uploadTempImg.do',
			type : 'post',
			data : form_data,
			contentType : false,
			processData : false,
			success : function(img_url){
				$(el).summernote('editor.insertImage', img_url);
				
				$('#event_pic_path').val(img_url.split('\$\$',2)[1]);
				$('#oldFileName').val(img_url.split('=',2)[1]);
			},
			error : function(err){
				console.log(err);
			}
			
		});
	}

	// 글 작성 도중 이미지를 지웠을 시에 temp폴더에서 지우기
	function deleteFile(src) {
		let splitSrc = src.split("=");
		let fileName = splitSrc[splitSrc.length - 1];
		
		let fileData = {
				fileName : fileName
		}
		
		console.log(fileData);
		console.log(JSON.stringify(fileData));
		
		$.ajax({
			url : "<%=request.getContextPath()%>/sysAdmin/deleteTempImg.do",
			type : "post",
			data : JSON.stringify(fileData),
			contentType : "application/json",
			success : function(res){
				$('#event_pic_path').val('');
				console.log(res);
			}
		});
	} 
```

```java
@RequestMapping("/common/uploadTempImg")
public ResponseEntity<String> uploadTempImg(MultipartFile file, HttpServletRequest req) {
  ResponseEntity<String> result = null;
  
  int fileSize = 10 * 1024 * 1024;
  
  if (file.getSize() > fileSize) {
    return new ResponseEntity<String>("용량 초과입니다.", HttpStatus.BAD_REQUEST);
  }
  
  String savePath = eventPicUploadPath + File.separator + "temp";
  String fileName = UUID.randomUUID().toString().replace("-", "")+"$$"+file.getOriginalFilename();
  System.out.println(fileName);
  File saveFile = new File(savePath, fileName);
  
  if(!saveFile.exists()) {
    saveFile.mkdirs();
  }
  
  try {
    file.transferTo(saveFile);
    result = new ResponseEntity<String>(req.getContextPath() + "/common/getTempImg.do?fileName=" + fileName, HttpStatus.OK);
  } catch (Exception e) {
    result = new ResponseEntity<String>(HttpStatus.INTERNAL_SERVER_ERROR);
  }
  
  return result;
}
 
@RequestMapping("/common/getTempImg")
public ResponseEntity<byte[]> getTempImg(String fileName, HttpServletRequest req) throws IOException {
  ResponseEntity<byte[]> entity = null;
  System.out.println(fileName);
  // 저장경로
  String savePath = eventPicUploadPath + File.separator + "temp";
  File sendFile = new File(savePath, fileName);
  
  InputStream in = null;
  
  try {
    in = new FileInputStream(sendFile);
    entity = new ResponseEntity<byte[]>(IOUtils.toByteArray(in), HttpStatus.CREATED);
  } catch (Exception e) {
    e.printStackTrace();
    entity = new ResponseEntity<byte[]>(HttpStatus.INTERNAL_SERVER_ERROR);
  } finally {
    in.close();
  }
  
  return entity;
}

@RequestMapping("/eventAdminDetail")
	public ModelAndView eventAdminDetail (ModelAndView mnv, String event_no, String type, HttpServletRequest req) throws NumberFormatException, SQLException {
		String url = "/sysAdmin/eventAdminDetail";
		
		if (event_no != null) {
			EventVO event = sysAdminService.selectEventByEvent_no(Integer.parseInt(event_no));
      // 이벤트 글 읽을 때 내용 안에 들어있는 이미지파일에 contextpath를 붙여줘야한다.
			event.setEvent_content(event.getEvent_content().replace("src=\"", "src=\""+req.getContextPath()));
			mnv.addObject("event", event);
		}
		mnv.addObject("type", type);
		
		List<Map<String, Object>> movieList = sysAdminService.selectMovieListForEventRegist();
		mnv.addObject("movieList", movieList);
		
		Map<String, Object> subjectMap = addSubject("HOME", "이벤트 관리", "진행중인 이벤트", url+".do?"+(event_no == null ? "" : "event_no="+event_no+"&")+"type="+type);
		mnv.addAllObjects(subjectMap);
		mnv.setViewName(url);
		return mnv;
	}
	
@RequestMapping("/eventAdminRegist")
  public void eventAdminRegist (EventRegistCommand registReq, HttpServletRequest req, HttpServletResponse res) throws SQLException, IllegalStateException, IOException {
    String eventPicUploadPath = this.eventPicUploadPath;
    EventVO event = registReq.toEventVO();
    System.out.println("event : " + event);
    
    // 이벤트 테이블에 등록
    sysAdminService.registEvent(event);
    int event_no = event.getEvent_no();
    
    // 이벤트 content안의 img url 수정
    String beforeSrcFormat = req.getContextPath() + "/common/getTempImg.do?fileName=";
    String modifiedEventContent = registReq.getEvent_content();
    
    if (registReq.getEvent_content().contains(beforeSrcFormat)) {	// 만약 이미지첨부를 했다면
      modifiedEventContent = registReq.getEvent_content().replace(beforeSrcFormat + registReq.getOldFileName() 
                                    ,"/common/getPicture.do?name="+registReq.getEvent_pic_path()+"&item_cd="+event_no+"&type=eventImg");
      Map<String, Object> modifyEventContentMap = new HashMap<>();
      modifyEventContentMap.put("event_no", event_no);
      modifyEventContentMap.put("newContent", modifiedEventContent);
      sysAdminService.modifyEventContent(modifyEventContentMap);
    }
    
    // 이벤트 썸네일 로컬에 저장
    MultipartFile thumb = registReq.getEvent_thum_path();
    if (thumb != null) {
      String fileName = thumb.getOriginalFilename();
      File target = new File(eventPicUploadPath + File.separator + event.getEvent_no() + File.separator + "thumb", fileName);
      
      if (!target.exists()) {
        target.mkdirs();
      }
      
      thumb.transferTo(target);
    }
    
    // 이벤트 이미지 temp폴더에서 알맞은 경로로 옮김
    if (registReq.getEvent_pic_path() != null && registReq.getEvent_pic_path() != "") {
      String fileName = event.getEvent_pic_path();
      File oldFile = new File(eventPicUploadPath + File.separator + "temp", registReq.getOldFileName());
      File newFilePath = new File(eventPicUploadPath + File.separator + event.getEvent_no() + File.separator + "img");
      if (!newFilePath.exists()) {
        newFilePath.mkdirs();
      }
      boolean renameTo = oldFile.renameTo(new File(newFilePath, fileName));
    }
    
    res.setContentType("text/html; charset=utf-8");
    PrintWriter out = res.getWriter();
    out.println("<script>");
    out.println("alert('이벤트 등록이 완료되었습니다.')");
    out.println("location.href='eventAdminDetail.do?type=read&event_no=" + event_no + "'");
    out.println("</script>");
    out.flush();
    out.close();
  }

// 주기적으로 temp폴더 비우기 (따로 util package 내 Scheduler Class에서 관리했음)
@Scheduled(cron = "0 0 4 ? * *")
public void removeTempImg() throws IOException {
  File directory = new File(eventPicUploadPath + File.separator + "temp");
  FileUtils.cleanDirectory(directory);
}
```
