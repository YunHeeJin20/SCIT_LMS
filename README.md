# SCIT_LMS
SCIT MASTER 학사관리 시스템
------------------
## 소개

> 일반 로그인과 카카오로그인 기능을 제공하며, 일본 IT 취업에 관한 정보를 제공한다.
> 과정에 참가한 사람들을 대상으로 하는 시스템이며, 연수생과 졸업생의 커뮤니케이션 
> 역할을 제공한다.
> 캘린더, 챗봇을 통한 간단한 질의응답, 일본의 IT기업에 대한 상세 정보, 잡페어와 잡페어에 
> 참가하는 기업정보, 센터의 스터디룸 예약관리, 공지 시 알림 기능을 구현하였다.
- [시연영상]( https://yunheejin20.github.io/heejin/WebContent/teamProject.html)
---------------
## 프로젝트 기획배경

> 여러 플랫폼을 사용하는 것이 아닌 하나의 플랫폼에서 공지, 잡페어정보, 수업정보, 취업정보를 > 제공하는 것에 대한 필요성을 느껴 학생관리시스템을 기획
---------------
## 사용기술

- JAVA,
- HTML,CSS
- JavaScript, JQuery, Ajax
- Spring, MyBati, Bootstrap
---------------
- FullCalendar
- Kakao 로그인 API, NaverClovaChabot API, Google Maps API
- WebSocket
---------------
## 개발환경

- Window OS
- Spring Tool Suite version : 4.3.6 RELEASE
- Tomcat 8.5
- Oracle SQL Developer 19.4
---------------
## 개인 역할
- 스터디룸 예약 관리 페이지 설계 및 디자인, 중복예약 확인 및 예약처리
- 스터디룸 예약 확인 페이지 설계 및 디자인, 퇴실,예약변경,예약취소 처리
- 일반회원, 카카오로그인회원 CRUD 처리
- 회원가입시 이미지 업로드, MY PAGE에서 회원정보 수정삭제 시 이미지도 함께 수정삭제 처리
- 잡페어 관리 CRUD처리
 ---------------
### 스터디룸별 에약가능시간 확인 
jsp
``` C
function timeList(){
         $.ajax({
            type:"post",
            url: "/roomBook/selectTime",
            data: {
               seat_sq : $('#seat_sq').val()
            },
            dataType: "json",
            success: function(res){
   
               var target = $('select[name="seat_aloc_strt_tm"]'); // 서브카테고리
               target.find('option').remove(); // 기존리스트 삭제

               $.each(res, function(index, item){
                  $('select[name="seat_aloc_strt_tm"]').append('<option value="' +item.timeLine+ '">' + item.timeLine + '</option>');
               });
               
            },
            error: function(e){
               alert("통신 실패");
               console.log(e);
            }
         });
      }
```

### 스터디룸 중복예약확인 처리과정
jsp
``` C
$(document).ready(function(){
         $("#doubleCheck").click(function(){
            $.ajax({
               url: "/roomBook/check",
               data: {
                  member_id : $("#user_id").val()
               },
               // dataType:"text",
               async: false,            // 비동기방식을 동기로 사용해서 true false 반환하기
               success: function(res){
                  if(res == "1"){
                     alert("Already occupied");
                      $('input[type="submit"]').attr("disabled", true);
                  }
                  else{
                     alert("Available");
                     $('input[type="submit"]').attr("disabled", false);
                  }
               },
               error: function(e){
                  alert("통신 실패");
                  console.log(e);
               }
      
               });
      
         });
      });
```

### 스터디룸 퇴실처리
Controller
``` C
@ResponseBody
      @RequestMapping(value="/roomEnd2", method=RequestMethod.POST)
      public String roomEnd2(RoomVO vo) {
         
         logger.info("전달받은 폼 데이터 : {}", vo.getSeat_aloc_sq());
         logger.info("전달받은 폼 데이터: {}", vo.getSeat_sq());
         logger.info("전달받은 폼 데이터 : {}", vo.getSeat_aloc_strt_tm());
         
         int seat_aloc_sq = vo.getSeat_aloc_sq();
         
         int seat_sq = vo.getSeat_sq(); // 퇴실하고 타임라인 0으로 업데이트
         String timeLine = vo.getSeat_aloc_strt_tm(); // 퇴실하고 타임라인 0으로 업데이트
               
         service.updateEndTm2(seat_aloc_sq); // 버튼누른 시간을 퇴실시간으로 업데이트
         
         service.timeLineCheck(seat_sq, timeLine); // 타임라인 0으로 업데이트
         
         RoomVO result = service.endTm(seat_aloc_sq); // 업데이트된 퇴실시간 검색을 위한 endTm 메소드호출
         String seat_aloc_end_tm = result.getSeat_aloc_end_tm(); 
         return seat_aloc_end_tm; 	// 검색한 퇴실시간을 return 한다
      }
```
### 개인 전체예약리스트 페이징
Controller
``` C
@RequestMapping(value="/roomCheck", method=RequestMethod.GET)
   public String checkBook(
         Model model,
         @RequestParam(value="page", defaultValue = "1") int page) {
      
      
      //퇴실안한 예약만 보여주기  위한 oneList 메소드 호출
      RoomVO list = service.oneList();
      model.addAttribute("listOne", list);
      
      // 예약변경을 위한 스터디룸 목록 조회 -- 수정 modal에 보여줄 것
      ArrayList<RoomVO> roomList = service.listSeat();   
      model.addAttribute("upList", roomList);
      
      // 페이징을 위한 예약수 조회를 위한 boardCount 메소드 호출
      int count = service.boardCount();
      
      // 페이징을 위한 pageNavigater 객체생성
      PageNavigator navi = new PageNavigator(COUNTPERPAGE, PAGEPERGROUP, page, count);
      
      //개인 예약 전체목록을 보여주기 위한 checkeBookOne 메소드 호출
      ArrayList<RoomVO> list2 = service.checkBookOne(navi.getStartRecord(), navi.getCountPerPage());
      model.addAttribute("seatList2", list2);	// 화면에 보여줄 리스트 데이터 model에 넣어주기
      
      model.addAttribute("navi", navi); // 페이징을 위해 모델에 넣어주기
      
      return "studyroom/checkReservationList";
   }
```
### 회원정보 수정처리
Controller 
``` C
@RequestMapping(value="/updateMember", method= {RequestMethod.GET, RequestMethod.POST})
      public String updateMember(MemberVO2 member, MultipartFile upload, HttpServletRequest req) {
         
         System.out.println("MemberController2 회원정보 수정 확인 : " + member);
         
       // 파일 수정
          if(!upload.isEmpty()) {
             //기존 파일 삭제
            String fullPath = uploadPath + "/" + member.getSavedfile();   
            boolean result = FileService.deleteFile(fullPath);
            
            // 파일전체경로 확인하기
            System.out.println("파일삭제를 위한 경로 : " + fullPath);
            // 삭제결과 확인하기
            System.out.println("파일삭제 여부 확인 : " + result);
             
            // 새로 첨부한 파일 등록
             String savedfile = FileService.saveFile(upload, uploadPath);
             member.setSavedfile(savedfile); // 물리적 저장파일명
             member.setOriginalfile(upload.getOriginalFilename()); 
          }else { 
             member.setSavedfile(req.getParameter("savedfile"));
             member.setOriginalfile(req.getParameter("originalfile"));
          }
         
         // 수정성공시 mypage로 이동
         String page = service2.updateMember(member);
         
         return page;
      }
```
