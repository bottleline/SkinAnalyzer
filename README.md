# SkinAnalyzer

## 주요기능

- C++ + opencv 를 사용하여 피부의 모공, 잡티, 주름을 검출 
- Tomcat 서버를 만들어 request 가 오면 분석 결과를 return

## 수행역할
- 설계 및 총 제작

## 사용한 기술
- `opencv`
- tomcat 8.0
- JSP

## 대략적인 구현 알고리즘
``` c
# 모공검출
	cv::cvtColor(img, img_after_median, COLOR_BGR2GRAY);                                          // 1. 메디안 필터
	histogramStreching(img_after_median, img_after_streching);                                    // 2 .히스토그램 스트레칭
	Laplacian(img_after_streching, img_after_laplacian, CV_8UC1);                                 // 3. 라플라시안 경계값 검출
	subtract(img_after_streching, img_after_laplacian, img_after_sub_edge);                       // 4. 스트레칭한 이미지 - 라플라시안 값
	inverseMedian = ~img_after_sub_edge;                                                          // 5. 이미지 반전 
	morphologyEx(inverseMedian, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);
	morphologyEx(image_after_tophat, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);      // 6. 블랙햇 연산

	morphologyEx(image_after_blackhat, image_after_blackhat, cv::MORPH_DILATE, line_row);         // 7. 세로 팽창 연산
	morphologyEx(image_after_blackhat, image_after_blackhat, cv::MORPH_DILATE, line_col);         // 8. 가로 팽창 연산
	
	morphologyEx(image_after_blackhat, image_after_blackhat, cv::MORPH_ERODE, line_row);          // 9. 가로 침식 연산
	morphologyEx(image_after_blackhat, image_after_blackhat, cv::MORPH_ERODE, line_col);          // 10 .세로 침식 연산

	cv::threshold(image_after_blackhat, image_after_blackhat, 10, 255, cv::THRESH_BINARY);        // 11. 스레시홀드 연산

	img_after_sub_blackhat = image_after_tophat - image_after_blackhat;                           // 12. 탑햇 - 블랙햇 이미지 

	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_row);                    // 13. 가로 침식 연산
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_col);                    // 14. 세로 침식 연산

	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_row);                    // 15. 가로 팽창 연산
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_col);                    // 16. 가로 팽창 연산
	

```  
``` c
# 잡티검출
self.adjust_gamma(img, 0.2)                     # 1. 감마값조절
cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)           # 2. 흑백화
self.adaptive_his_streching(img)                # 3. 어댑티브 히스토그램 스트레칭 연산
cv2.medianBlur(_, _)                            # 4. 메디안 필터 연산
cv2.morphologyEx(_, cv2.MORPH_BLACKHAT, kernal) # 5. 블랙햇 연산
cv2.threshold(_, _, _, cv2.THRESH_BINARY)       # 6. thresh hold 값에 따른 이진화
self.check_eccen3(bt, 20, 1200, 0.4)            # 7. 타원형체크 (결과값이 원형에 가까운지 검출)
```  
``` c
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

``` java
# tomcat server

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

