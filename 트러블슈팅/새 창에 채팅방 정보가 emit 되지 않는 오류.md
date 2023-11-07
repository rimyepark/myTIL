# 새 창에 채팅방 정보가 emit 되지 않는 오류

## 문제 상황
 수정 API 화상채팅방 오른쪽 상단의 나가기 버튼에 연결하였다. 나가기 버튼을 누르면 채팅방 상태가 채팅중에서 나가기로 변경되어야한다. 하지만 첫 테스트를 할때 버튼을 눌렀으나 로직이 실행되지 않았다.


 ## 근본 원인

 console.log 을 찍어보니 채팅방의  facechatId 값이 null 값으로 나왔다. 왜 이러는지 확인하기 위해서 draw.js 와 socket.js 서버내에 각각 로직마다 console.log을 찍어보았다. 채팅을 초대할때 까지는 룸 번호가 계속해서 서버로 전달되었는데 채팅창의 새창이 뜨는 순간  facechatId 값이 사라졌다. 새 창 까지 emit 이 되지 않는 현상을 찾았다.

## 해결 방법

새 창까지  facechatId 값을 전달하기 위해 저장하는 방법을 생각했다. 저장의 방법은 총 두가지. 쿠키와 로컬스토리지다. 

localStorage: 브라우저 세션 간에 데이터를 유지해야하고, 데이터의 용량이 쿠키보다 크며, 서버와의 각 요청에서 데이터를 보낼 필요가 없다면, 로컬 스토리지가 적합할 수 있다.

Cookies: 서버와 클라이언트 사이에서 데이터를 주고받아야 하거나, 서브도메인 간에 데이터를 공유해야 한다면 쿠키가 적합할 수 있다. 또한, 서버에서 HTTP 응답을 통해 클라이언트에 데이터를 설정할 필요가 있다면 쿠키를 사용해야 한다.

 위의 사례를 봤을 때 쿠키를 사용하는 것이 더 적합하다고 판단하였으나 일단 로직이 구현되는 것을 확인하기 위해 몇줄만 추가해도 되는 로컬스토리지를 선택하였다.

```javascript
if (response.status === 200 || response.status === 201) {
    const responseData = await response.json();
    facechatId = responseData.facechat_id;
    console.log('API POST successful');
    
    // 값을 localStorage에 저장
    localStorage.setItem('facechatId', facechatId);

    socket.emit('accept_face_chat', inviterId, currentUserId, roomId);
}
```


![로컬스토리지 저장](https://github.com/rimyepark/myTIL/assets/133215556/700a3574-7caa-42f1-815d-1cd2e1073e58)


 이후 다시 나가기 버튼을 눌렀다. 수정 api 는 성공적으로 실행이 되었고 채팅방 상태가 채팅중에서 나가기로 변경이 되었다.

 ![나가기성공](https://github.com/rimyepark/myTIL/assets/133215556/7796115b-038d-472e-ad08-6b16d56a8fae)

이후 로직을 쿠키로 변경해서 재구동하니 마찬가지로 성공하였다.
