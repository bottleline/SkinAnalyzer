# SkinAnalyzer_Python

## 주요기능

- python3.8 + opencv 를 사용하여 피부의 모공, 잡티, 주름을 검출 
- flask 서버를 만들어 request 가 오면 분석 결과를 return

## 수행역할
- 설계 및 총 제작

## 사용한 기술
- `cv2`, `dlib`, `flask`

## 대략적인 구현 알고리즘
``` python
# 모공검출

self.adjust_gamma(img,0.2)                      # 1.감마값조절
cv2.cvtColor(_, cv2.COLOR_BGR2GRAY)             # 2.그레이체널화
self.adaptive_his_streching(_)                  # 3.어댑티브 히스토그램 명암대비
cv2.Laplacian(_, cv2.CV_8U, ksize=_)            # 4. 라플라시안 경계값 검출
cv2.subtract(_, _)                              # 5.스트레칭값 - 라플라시안 값
cv2.bitwise_not(_)                              # 6.이미지 반전
cv2.morphologyEx(_, cv2.MORPH_TOPHAT, kernel)   # 7. 탑햇연산
cv2.morphologyEx(_, cv2.MORPH_BLACKHAT, kernel) # 8. 블랙햇연산
cv2.dilate(_,_,iterations=1)                    #
cv2.erode(_,_,iterations=1)                     # 9. 열림연산
cv2.threshold(_, _, _, cv2.THRESH_BINARY)       # 10. 이진화
self.check_eccen3(thresh2, 1, 200, 0.7)         # 11. 타원형체크 (결과값이 원형에 가까운지 검출)

```  
``` python
# 잡티검출
self.adjust_gamma(img, 0.2)                     # 1. 감마값조절
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)           # 2. 흑백화
self.adaptive_his_streching(img)                # 3. 어댑티브 히스토그램 스트레칭 연산
cv2.medianBlur(_, _)                            # 4. 메디안 필터 연산
cv2.morphologyEx(_, cv2.MORPH_BLACKHAT, kernal) # 5. 블랙햇 연산
cv2.threshold(_, _, _, cv2.THRESH_BINARY)       # 6. thresh hold 값에 따른 이진화
self.check_eccen3(bt, 20, 1200, 0.4)            # 7. 타원형체크 (결과값이 원형에 가까운지 검출)
```  
``` python
# 주름검출
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)           # 1. 흑백화
self.adaptive_his_streching(img)                # 2. 어댑티브 히스토그램 스트레칭 연산
cv2.morphologyEx(_, cv2.MORPH_TOPHAT, kernel)   # 3. 탑햇 연산
cv2.medianBlur(tophat, _)                       # 4. 메디안 필터 연산
self.detect_ridges(median, sigma=1.0)           # 5. 릿지검출 연산
cv2.threshold(_, thresh, 255, cv2.THRESH_BINARY)# 6. thresh hold 값에 따른 이진화
self.check_eccen3(ridge_thresh, 10, 500, eccen) # 7. 주름성 체크 (원형에 가까운 모양인지 선형에 가까운 모양인지 체크)
skeletonize(result, method='lee')               # 8. 스켈레토나이즈 연산

```

``` python
# flask server

p = pore()     # 모공검출 객체생성
pi = pigment() # 잡티검출 객체생성
w = wrinkle()  # 주름검출 객체생성

.
.
.
.

return json.dumps(result_dict, ensure_ascii=False, indent="\t", cls=NpEncoder) 
# 결과 이미지를 base64로 인코딩 후 결과 값과 함께 json 형태로 return

```
## 결과 화면 예시

### 모공
![image](https://user-images.githubusercontent.com/42457589/142145977-762dd5d3-59ef-4235-b798-904b579898de.png)
  
### 주름
![image](https://user-images.githubusercontent.com/42457589/142145853-b3d502f0-d94e-4758-b506-1f3b732f538f.png)
  
### 잡티
![image](https://user-images.githubusercontent.com/42457589/142146069-4154dc9d-39d1-4cd1-b01e-a9e4d72d64a2.png)

