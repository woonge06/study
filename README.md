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
```
만약 이미지를 확대/축소하려면, GDI+를 사용하거나, 이미지를 리샘플링하는 방법을 사용할 수 있습니다. 기본적으로 MFC는 StretchBlt 함수나 CDC::DrawState 등을 사용하여 이미지를 그릴 수 있습니다.
예를 들어, StretchBlt를 사용하여 비트맵을 확대/축소할 수 있습니다.
``` ruby
CDC* pDC = GetDC();
CImage image;
image.Load(_T("image.bmp")); //이미지 파일 로드
int newWidth = image.GetWidth() * zoomFactor;
int newHeight = image.GetHeight() * zoomFactor;
image.StretchBlt(pDC->m_hDC, 0, 0, newWidth, newHeight, 0, 0, image.GetWidth(), image.GetHeight(), SRCCOPY);
ReleaseDC(pDC);
```
성능 최적화 : 확대/축소 기능을 구현할 때 성능을 고려해야 합니다. 이미지가 커지거나 복잡한 도형이 많을 경우, 오버드로잉(중복 그리기) 문제를 피하기 위해 더블 버퍼링을 사용할 수 있습니다. 또한, 확대/축소 후 화면 갱신 시 불필요한 계산을 줄여 성능을 최적화하는 것도 중요합니다.

-----------------------
mfc에서 가상의 좌표는 주로 논리 좌표 개념을 기반으로 한 것입니다. 논리 좌표는 개발자가 설정한 사용자 정의 좌표계로, 실제 화면의 픽셀 좌표(장치 좌표, Device Coordinates)와는 별개의 추상적인 좌표 체계입니다.
이 좌표 개념은 화면의 확대/축소, 스크롤, 회전, 그리고 다양한 좌표 변환을 간단하고 직관적으로 처리할 수 있도록 설계되었습니다.
장치 좌표 : 실제 화면(또는 출력 장치)에서의 픽셀 단위 좌표입니다.
ex) 해상도 1920x1080 디스플레이에서는 (0, 0)부터 (1919, 1079)까지가 장치 좌표입니다.
화면의 왼쪽 상단 (0, 0)을 기준으로 하며, 오른쪽으로 x값이 증가, 아래쪽으로 y값이 증가합니다.
논리 좌표 : 사용자가 정의한 추상적인 좌표계입니다.
논리 좌표는 반드시 장치 좌표와 일치할 필요가 없으며, 프로그래머가 원하는 대로 설정 가능합니다.
논리 좌표를 장치 좌표에 변환할 때, 맵 모드를 설정하고, 비율과 방향 등을 제어합니다.
맵 모드 : 맵 모드는 논리 좌표를 장치 좌표로 변환하는 방식(스케일, 단위, 방향 등)을 정의합니다. mfc에서 CDC::SetMapMode를 사용하여 설정합니다. 주요 맵 모드와 특징은
* 고정 비율 맵 모드
  1. MM_TEXT : 논리 좌표 == 장치 좌표 (1:1 비율), 픽셀 단위로 좌표를 설정하며, 가장 기본적인 맵 모드입니다. (ex : (10, 10) 논리 좌표는 (10px, 10px)로 직접 변환됩니다.)
  2. MM_LOMETRIC : 1mm에 해당하는 논리 좌표는 10 단위(ex : 논리 좌표 (100, 100)은 10mm의 크기로 장치 좌표로 변환됩니다.)
  3. MM_HIMETRIC : 1mm에 해당하는 논리 좌표는 100 단위로 보다 정밀한 크기 조정이 가능합니다.
  4. MM_LOENGLISH / MM_HIENGLISH : 논리 좌표 단위를 인치로 변환합니다.
* 사용자 정의 비율 맵 모드
  1. MM_ISOTROPIC : 논리 좌표의 x축과 y축 비율을 동일하게 설정합니다, x축과 y축의 비율이 동일해야 하므로, 화면 비율을 유지한 채 확대/축소 가능합니다.
  2. MM_ANISOTROPIC : x축과 y축의 비율을 독립적으로 설정할 수 있습니다, 논리 좌표를 장치 좌표로 변환할 때, 다른 비율을 사용할 수 있어 자유도가 높습니다.
* 논리 좌표와 장치 좌표의 변환 : 논리 좌표를 장치 좌표로 변환하기 위해 다음과 같은 설정이 필요합니다
1. SetMapMode : 맵 모드를 설정하여 논리 좌표계를 정의합니다. (dc.SetMapMode(MM_ANISOTROPIC); // 사용자 정의 비율로 설정)
2. SetWindowExt : 논리 좌표계의 크기를 설정합니다. 이는 논리 좌표의 범위를 결정합니다. (dc.SetWindowExt(100, 100); // 논리 좌표계를 100x100 범위로 설정)
3. SetViewportExt : 장치 좌표계의 크기를 설정합니다. 이는 논리 좌표를 장치 좌표로 변환할 때 사용할 스케일을 결정합니다. (dc.SetViewportExt(200, 200); // 장치 좌표계를 200x200 범위로 설정)
장치 좌표는 수식을 통해 계산됩니다. 수식은 장치좌표 = 논리 좌표 X Viewport Extent/Window Extent
ex) 논리 좌표(50, 50), SetWindowExt(100, 100), SetViewportExt(200, 200) -> 계산 결과 : 장치 좌표 = (50\times \frac{200}{100}, 50 \times \frac{200}{100}) = (100, 100)
4. SetViewportOrg와 SetWindowOrg : 이 두 함수는 좌표계의 원점을 설정합니다. 기본적으로 원점은 (0, 0)이지만, 원하는 좌표로 이동할 수 있습니다.
``` ruby
dc.SetViewportOrg(50, 50); //장치 좌표 원점을 (50, 50)으로 이동
dc.SetWindowOrg(-10, -10); //논리 좌표 원점을 (-10, -10)으로 이동
```
* 가상의 좌표 활용 예제
확대/축소 예제 : SetWindowExt와 SetViewportExt를 적절히 설정합니다.
``` ruby
void CMyView::OnPaint()
{
  CPaintDC dc(this); //DC 생성
  //맵 모드 설정
  dc.SetMapMode(MM_ANISOTROPIC);
  //논리 좌표와 장치 좌표 설정
  float zoomFactor = 2.0f; //확대 비율(2배 확대)
  dc.SetWindowExt(100, 100); //논리 좌표계
  dc.SetViewportExt(100 * zoomFactor, 100 * zoomFactor); //장치 좌표계
  //논리 좌표로 사각형 그리기
  dc.Rectangle(10, 10, 90, 90);
}
```
스크롤 구현 예제 : 논리 좌표 원점을 변경해야 합니다. 이를 위해 SetWindowOrg를 사용합니다.
``` ruby
void CMyView::OnScroll(int scrollX, int scrollY)
{
  CPaintDC dc(this);
  dc.SetMapMode(MM_ANISOTROPIC);
  //스크롤에 따른 논리 좌표 원점 이동
  dc.SetWindowOrg(scrollX, scrollY);
  //다시 그리기
  dc.Rectangle(10, 10, 90, 90);
}
```
* mfc 가상의 좌표 장점
  1. 확장성 : 화면 크기, 해상도에 상관없이 동일한 논리 좌표로 작업 가능
  2. 확대/축소 : 비율만 변경하면 간단히 구현 가능
  3. 회전 및 변형 : 원점 및 좌표계 설정을 통해 복잡한 변환도 처리 가능

----------------
