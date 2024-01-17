## 첫번째. add, edit, create와 같은 단어를 쓰지 않는다.
![[스크린샷 2024-01-17 오전 10.56.36.png]]

**변경 후** 
![[스크린샷 2024-01-17 오전 10.57.13.png]]

동사를 없앤다면 URL에는 명사 밖에 남지 않습니다. 

#### 그렇다면 복수와 단수를 어떻게 구분할까?

GET /movies/{movieId}
GET /movies

위 두 개의 차이가 느껴진다면 성공입니다.

## 두번째. 다른 좋은 예시
/getTopRatedMovies  =  /movies?min_rating=9.8
/findMoviesFromThisYear = /movies?release_date=2021

## Reference
- https://www.youtube.com/watch?v=4DxHX95Lq2U