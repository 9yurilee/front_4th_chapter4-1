# 🚀 프론트엔드 배포 파이프라인

## 📝 개요
본 프로젝트는 **GitHub Actions을 활용하여 AWS S3 및 CloudFront를 기반으로 Next.js 프로젝트를 자동 배포하는 CI/CD 파이프라인**을 구축하였습니다.  

---

## 📌 배포 흐름 (CI/CD Pipeline)

1. **코드 변경 사항 발생** (`git push` 또는 PR 생성)
2. **GitHub Actions 실행**
3. **코드 체크아웃** (`Checkout Repository`)
4. **의존성 설치** (`npm ci`)
5. **Next.js 프로젝트 빌드** (`npm run build`)
6. **AWS IAM 자격 증명 설정** (`GitHub Secrets`)
7. **빌드된 파일을 S3 버킷에 업로드** (`aws s3 sync`)
8. **CloudFront 캐시 무효화** (`aws cloudfront create-invalidation`)
9. ✅ **배포 완료 🎉**

---

## 📌 상세 배포 과정
![배포 인프라 다이어그램](https://github.com/user-attachments/assets/5a049059-22ba-449d-a525-554edf91bb1e)

1. **개발자가 GitHub 저장소에 코드 푸시 (`git push`)**
2. **GitHub Actions 실행**
   - `npm ci`를 통해 의존성을 설치
   - `npm run build`로 프로젝트를 빌드
3. **AWS S3에 배포 (정적 파일 업로드)**
   - 빌드된 정적 파일을 S3에 업로드 (`aws s3 sync`)
4. **CloudFront를 통한 글로벌 배포**
   - CloudFront가 S3에서 최신 파일을 가져옴
   - 캐시 무효화를 수행하여 즉시 최신 콘텐츠 제공
5. **사용자는 CloudFront 도메인을 통해 웹사이트에 접속**  

---

## 🌐 주요 링크
- **S3 버킷 웹사이트 엔드포인트:** http://hanghae41.s3-website.us-east-2.amazonaws.com/
- **CloudFront 배포 도메인:** https://d29p92crmmop5c.cloudfront.net

---

## 📌 주요 개념

### 🔹 GitHub Actions과 CI/CD 도구
- **GitHub Actions**는 GitHub에서 제공하는 CI/CD(Continuous Integration & Continuous Deployment) 서비스로,  
  코드를 푸시할 때 자동으로 빌드, 테스트, 배포를 수행할 수 있도록 도와줍니다.
- CI/CD는 **개발 프로세스를 자동화하여 코드 품질을 유지하고 배포 속도를 높이는 방법론**입니다.
- 이 프로젝트에서는 **GitHub Actions를 이용하여 Next.js 프로젝트를 빌드하고 AWS S3 및 CloudFront에 배포**합니다.

### 🔹 S3와 스토리지
- **Amazon S3 (Simple Storage Service)**는 AWS에서 제공하는 **클라우드 객체 저장소**로,  
  정적 웹사이트 호스팅 및 다양한 데이터 저장 용도로 사용됩니다.
- Next.js 프로젝트의 빌드 결과물(HTML, CSS, JavaScript 파일 등)을 S3 버킷에 업로드하여 배포합니다.
- S3는 높은 내구성과 가용성을 제공하며, CloudFront와 함께 사용하면 성능을 더욱 향상시킬 수 있습니다.

### 🔹 CloudFront와 CDN
- **Amazon CloudFront**는 AWS의 **콘텐츠 전송 네트워크(Content Delivery Network, CDN)** 서비스입니다.
- CloudFront는 **전 세계 엣지 로케이션(Edge Location)에 콘텐츠를 캐싱하여 성능을 최적화**하고,  
  사용자에게 더 빠르게 웹사이트를 제공할 수 있도록 도와줍니다.
- 이 프로젝트에서는 **CloudFront가 S3에서 정적 파일을 가져와 사용자에게 제공**하는 역할을 합니다.

### 🔹 캐시 무효화 (Cache Invalidation)
- **CloudFront는 캐싱을 통해 성능을 최적화하지만, 새로운 배포가 이루어지면 최신 파일을 즉시 반영해야 합니다.**
- 이를 위해 **CloudFront 캐시를 무효화(Invalidation)하여 변경된 파일이 즉시 반영되도록 설정**합니다.
- GitHub Actions에서 **다음 명령어를 실행하여 배포 후 캐시를 삭제**합니다.

  ```sh
  aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
