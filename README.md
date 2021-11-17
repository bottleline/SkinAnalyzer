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
// 모공검출
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
// 잡티검출

	cv::cvtColor(img, img_after_median, COLOR_BGR2GRAY); 						// 1. 그레이스케일
	cv::medianBlur(img_after_median, img_after_median, 3); 						// 2. 메디안필터
	histogramStreching(img_after_median, img_after_streching);					// 3. 히스토그램 스트레칭
	inverseMedian = ~img_after_streching;								// 4. 이미지 반전
	morphologyEx(inverseMedian, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);	
	morphologyEx(image_after_tophat, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);	// 5. 탑햇
	cv::medianBlur(img_after_median, img_after_median, 3);						// 6. 메디안 필터
	histogramStreching(img_after_median, img_after_streching);					// 7. 히스토그램 스트레칭
	cv::GaussianBlur(img_after_median, img_after_gaussian, Size(5, 5), 1.5);			// 8. 가우시안 블러
	inverseMedian = ~img_after_streching;								// 9. 이미지 반전

	morphologyEx(inverseMedian, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);
	morphologyEx(image_after_tophat, image_after_tophat, cv::MORPH_TOPHAT, circle_blackhat);	// 10. 탑햇 연산

	cv::threshold(image_after_tophat, img_after_thresh, 55, 255, cv::THRESH_BINARY);		// 11. 스레시홀드 연산

	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_row);			
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_col);

	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_row);
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_col);			// 10. 닫힘 연산
	
```  
``` c
// 주름검출

	cv::cvtColor(cl, cl, COLOR_GRAY2BGR);									// 1. 그레이스케일
	cl.convertTo(cl, CV_8UC3);										// 2. 3채널화
	cv::GaussianBlur(cl, img_after_gaussian, Size(5, 5), 1.5);						// 3. 가우시안 블러
	garborFilter(img_after_gaussian, image_after_gaborfilter);						// 4. 가버필터 적용
	RDF = ximgproc::RidgeDetectionFilter::create(CV_32FC1, 1, 1, 3, CV_8UC1, 3, 0, BORDER_DEFAULT);		
	RDF->getRidgeFilteredImage(image_after_gaborfilter, image_after_ridge_detection);			// 5. 릿지 디텍션 필터
	cv::threshold(image_after_ridge_detection, image_after_ridge_detection, tv, 255, cv::THRESH_BINARY);	// 6. 스레시홀드 연산
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_row);
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_ERODE, line_col);
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_row);
	morphologyEx(img_after_morph, img_after_morph, cv::MORPH_DILATE, line_col);				// 11. 열기 연산
	checkEccentricity(img_after_morph, type, WRIN);								// 12. 타원형 모양 체크
	cv::threshold(com, com, 0, 255, cv::THRESH_BINARY);							// 13. 스레시홀드 연산
```

## 결과 화면 예시

### 모공
![image](https://user-images.githubusercontent.com/42457589/142155567-f81fccad-9ace-42f6-b1d9-93b1ff1afdd9.png)
  
### 주름
![image](https://user-images.githubusercontent.com/42457589/142155520-6fb49516-66f9-4421-a69f-1e35cc88c2b5.png)
  
### 잡티
![image](https://user-images.githubusercontent.com/42457589/142155602-a0f31041-df69-484b-ba45-363ae425251c.png)

