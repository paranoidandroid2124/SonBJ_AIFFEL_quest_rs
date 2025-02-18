# AIFFEL Campus Online Code Peer Review Templete
- 코더 : 손병진
- 리뷰어 : 조현철


# PRT(Peer Review Template)
- [✓ ]  **1. 주어진 문제를 해결하는 완성된 코드가 제출되었나요?**
![image](https://github.com/user-attachments/assets/b4d6d044-dc39-4386-93a5-e1bbe00f1a16)

- 
    - 문제에서 요구하는 최종 결과물이 첨부되었습니다.
    
- [ ✓]  **2. 전체 코드에서 가장 핵심적이거나 가장 복잡하고 이해하기 어려운 부분에 작성된 
주석 또는 doc string을 보고 해당 코드가 잘 이해되었나요?**
![image](https://github.com/user-attachments/assets/42f01ee9-d195-4326-8680-5790b67dda8e)

        - 얼굴 랜드마크 기반의 정밀한 스티커 배치와 자연스러운 합성을 코드를 통해 알 수 있었습니다 
        
- [✓ ]  **3. 에러가 난 부분을 디버깅하여 문제를 해결한 기록을 남겼거나
새로운 시도 또는 추가 실험을 수행해봤나요?**
    - 각도 전환, 전신과 얼굴사진 등의 비교를 통해서 임계점을 알아보려는 새로운 시도를 하였습니다.
        
- [✓ ]  **4. 회고를 잘 작성했나요?**
![image](https://github.com/user-attachments/assets/6a5b1842-7db3-49da-82d7-fa6182fa3f14)

회고를 상세히 잘 작성했습니다.
        
- [✓ ]  **5. 코드가 간결하고 효율적인가요?**
 ![image](https://github.com/user-attachments/assets/30b0d350-efa6-4dc7-97e0-acdeda56f6f4)

코드는 주석이 잘 달려있고 간결하고 효율적입니다.

# 회고(참고 링크 및 코드 개선)
- 테스트 1. 온진하지 않는 얼굴(일부만 나온 경우)
    - 얼굴 전체가 아닌 일부만 나온 이미지에서 정상적으로 얼굴 감지가 되는지 테스트
    - 테스트 결과
        - 얼굴의 일부만 나온 경우 얼굴 자체를 인식하지 못함

- 테스트 2. 얼굴이 기울어진 경우(회전된 얼굴)
    - 얼굴이 기울어져 있을 때 얼굴 박스와 랜드마크가 정상적으로 감지되는지 확인
    - 테스트 결과
        - 얼굴 전체가 나온 경우 라운딩 박스는 정상적으로 탐지됨
        - 눈, 코, 입 위치 값이 변경되면서 랜드마크 키포인트는 정상적으로 탐지되지 않음
- 테스트 2-1. 회전 임계값 찾기(얼마나 회전하면 감지 실패하는지)
    - 5~70도까지 5도 간격으로 이미지를 회전시키면서 어느 각도까지 정상적으로 얼굴이 감지되는지 테스트
    - 테스트 결과
        - 25도까지는 눈,코,입 비교적 정확하게 탐지
        - 30도부터는 랜드마크 위치를 정확히 탐지하지 못함

- 테스트 3. 얼굴 크기에 따른 인지(얼굴사진 vs 전신사진)
    - 얼굴이 강조된 사진 4장과 전신 사진 4장을 비교하여 얼굴 크기가 감지 성능에 미치는 영향을 확인
    - 테스트 결과
        - 두 그룹 다 정상적으로 작동하지 않음
        - 원인 1. 이미지 해상도문제
            - 이미지가 너무 크거나 작은 경우 감지율이 떨어질 수 있음
        - 원인 2. 조명, 보정 문제
            - 수집된 이미지는 화보사진
            - 일반적인 셀카와 다르게 피부 보정, 강한 조명 적용
            - 피부결이 일정해지거나 눈, 코, 입의 경계가 약해지면서 얼굴 감지가 어려움.

- 테스트 결론
    - 모델의 한계 
        - 얼굴이 부분적으로 나오거나 회전되거나 보정이 강한 사진인 경우 감지 성능이 떨어짐
    - 추가 테스트 필요
        - 화보 사진과 같은 경우 경계를 강조하는 전처리 필요
        - 해상도나 밝기 별 추가적인 테스트 필요

```
실험 시 여러장의 사진을 핸들링하기 위해서 이미지 list를 입출력으로 받는 함수를 만들어서 사용

# 라운딩박스 및 랜드마크 세팅 함수
def draw_faces_and_landmarks(image_list, dlib_rects, landmark_predictor):
    processed_images = []

    for img in image_list:
        img_copy = img.copy() 

        for dlib_rect in dlib_rects:
            # 얼굴 박스 그리기 (초록색)
            l, t, r, b = dlib_rect.left(), dlib_rect.top(), dlib_rect.right(), dlib_rect.bottom()
            cv2.rectangle(img_copy, (l, t), (r, b), (0, 255, 0), 10, lineType=cv2.LINE_AA)

            # 랜드마크 검출
            points = landmark_predictor(img_copy, dlib_rect)
            list_points = [(int(p.x), int(p.y)) for p in points.parts()]

            # 랜드마크를 노란색 (0, 255, 255) 원으로 표시
            for point in list_points:
                cv2.circle(img_copy, point, 20, (0, 255, 255), -1)  # 크기 3, 노란색 원

        processed_images.append(img_copy)

    return processed_images

# 여러장 사진을 출력하는 함수
def show_images(image_list):
    num_images = len(image_list)

    if num_images == 0:
        print("이미지 리스트가 비어 있습니다.")
        return
    
    rows = math.ceil(num_images / 2)
    
    plt.figure(figsize=(10, rows * 3))
    
    for idx, img in enumerate(image_list):
        plt.subplot(rows, 2, idx + 1)
        plt.imshow(cv2.cvtColor(img, cv2.COLOR_BGR2RGB)) 

    plt.tight_layout()
    plt.show()

```
