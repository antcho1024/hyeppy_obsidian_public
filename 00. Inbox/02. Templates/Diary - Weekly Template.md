<%* let year = tp.file.title.split('-W')[0]; 
let week = tp.file.title.split('-W')[1]; 
let mondayDate = moment().year(year).week(week).startOf('isoWeek'); // 주의 시작을 월요일로 설정

// 요일 배열 생성 
let weekdays = []; for (let i = 0; i < 7; i++) { 
	weekdays.push(mondayDate.clone().add(i, 'days').format('YYYY-MM-DD (ddd)')); 
	} -%> 
	
#### 회고
---
##### 월요일 [[<% weekdays[0] %>]]


##### 화요일 <% weekdays[1] %> 


##### 수요일 <% weekdays[2] %> 


##### 목요일 <% weekdays[3] %> 


##### 금요일 [[<% weekdays[4] %>]]


##### 토요일 <% weekdays[5] %> 


##### 일요일 <% weekdays[6] %>