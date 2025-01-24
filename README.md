# study
mfc로 만든 그림판 프로그램에서 확대, 축소 기능을 구현하기 위해선 좌표 반환과 렌더링 방식을 이해해야 합니다.

---------------------------
확대,축소는 실제 이미지 데이터는 그대로 두고, 화면에 표시되는 좌표계를 변환하여 구현하는데,
장치 좌표 : 화면에 직접 그려지는 좌표계로 픽셀 단위입니다.
논리 좌표 : 프로그램에서 사용하는 가상 좌표입니다.
mfc에서는 SetMapMode, SetViewportExt, SetWiindowExt 함수를 사용해 논리 좌표와 장치 좌표 간의 변환을 설정할 수 있습니다.
확대/축소를 위한 좌표 변환
SetMapMode, SetViewportExt, SetWindowExt 메서드를 사용해 처리할 수 있습니다.
먼저, 맵 모드를 설정해야 합니다. MM_ANISOTROPIC 모드를 사용하면 논리 좌표와 장치 좌표 사이의 비율을 자유롭게 조정할 수 있습니다.
``` ruby
dc.SetMapMode(MM_ANISOTROPIC); //비율을 자유롭게 설정할 수 있는 모드
```
SetWindowExt : 논리 좌표의 범위를 설정합니다., SetViewportExt : 장치 좌표의 범위를 설정합니다.
``` ruby
dc.SetWindowExt(100, 100); //논리 좌표(100, 100)
dc.SetViewportExt(200, 200); //장치 좌표(200, 200) -> 2배 확대
```
확대/축소 비율은 zoomFactor라는 변수로 관리할 수 있습니다. zoomFactor가 1이면 원래 크기, 2면 2배 확대, 0.5면 2배 축소된 상태입니다.
``` ruby
float zoomFactor = 2.0; //2배 확대
dc.SetWindowExt(100, 100); //논리 좌표
dc.SetViewportExt(100 * zoomFactor, 100 * zoomFactor); //장치 좌표
```
마우스 휠 이벤트를 통해 실시간으로 확대/축소 비율을 변경할 수 있습니다. mfc에서는 이를 WM_MOUSEWHEEL 메시지를 사용하여 이를 처리합니다.
WM_MOUSEWHEEL 메시지는 마우스 휠이 돌아갈 때 발생합니다. 이 메시지를 처리하여 zoomFactor값을 조정합니다.
``` ruby
BOOL CMyView::OnMouseWheel(UNIT nFlags, short zDelta, CPoint pt)
{
  if(zDelta > 0)
  {
    zoomFactor += 0.1;
  }
  else if(zDelta < 0)
  {
    zoomFactor = max(0.1, zoomFactor - 0.1);
  }
  //확대,축소 후 화면 갱신
  Invalidate();
  return CView::OnMouseWheel(nFlags, zDelta, pt);
}
```
