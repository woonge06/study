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
  if(zDelta > 0) //휠 업: 확대
  {
    zoomFactor += 0.1;
  }
  else if(zDelta < 0) //휠 다운: 축소
  {
    zoomFactor = max(0.1, zoomFactor - 0.1);
  }
  //확대,축소 후 화면 갱신
  Invalidate();
  return CView::OnMouseWheel(nFlags, zDelta, pt);
}
```
Invalidate() 함수는 화면을 갱신하여, 확대/축소된 내용을 다시 그리도록 합니다. 이때, OnPaint 함수에서 다시 그리기를 할 수 있도록 합니다.
``` ruby
void CMyView::OnPaint()
{
  CPaintDC dc(this); //디바이스 컨텍스트 가져오기
  dc.SetMapMode(MM_ANISOTROPIC); //비율 설정
  dc.SetWindowExt(100, 100); //논리 좌표
  dc.SetViewportExt(100 * zoomFactor, 100 * zoomFactor); //장치 좌표
  //예시로 사각형 그리기
  dc.Rectangle(10, 10, 90, 90);
}
```
확대/축소 시 화면의 중심이 달라지는 문제를 해결하려면, 확대/축소 비율에 맞게 화면을 재조정해야 합니다. 이를 위해 스크롤 뷰를 활용하거나, 중심 좌표를 고정한 상태로 뷰를 갱신하는 방법을 사용할 수 있습니다.
CScrollView는 스크롤 기능을 제공하며, 확대/축소 시 스크롤 바를 통해 화면을 이동할 수 있도록 도와줍니다. 확대/축소 비율이 변하면 스크롤 영역도 함께 조정되어 화면의 중심을 유지할 수 있습니다.
``` ruby
void CMyView::OnInitialUpdate()
{
  CScrollView::OnInitialUpdate();
  //논리 좌표와 장치 좌표 설정
  SetScrollSizes(MM_TEXT, CSize(100 * zoomFactor, 100 * zoomFactor));
}
만약 이미지를 확대/축소하려면, GDI+를 사용하거나, 이미지를 리샘플링하는 방법을 사용할 수 있습니다. 기본적으로 MFC는 StretchBlt 함수나 CDC::DrawState 등을 사용하여 이미지를 그릴 수 있습니다.
예를 들어, StretchBlt를 사용하여 비트맵을 확대/축소할 수 있습니다.
``` ruby
CDC* pDC = GetDC();
CImage image;
image.Load(_T("image.bmp"));
int newWidth = image.GetWidth() * zoomFactor;
int newHeight = image.GetHeight() * zoomFactor;
image.StretchBlt(pDC->m_hDC, 0, 0, newWidth, newHeight, 0, 0, image.GetWidth(), image.GetHeight(), SRCCOPY);
ReleaseDC(pDC);
```
