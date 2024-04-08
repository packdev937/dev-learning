```
name: CD

on:
  push:
    branches: [develop]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Permission for Gradle Wrapper
        run: |
          cd team-g
          chmod +x ./gradlew

      - name: Build with Gradle
        run: |
          cd team-g
          ./gradlew build -x test
        shell: bash

      - name: Docker Build & Push
        run: |
          docker login -u ${{ secrets.DOCKER_ID }} -p ${{ secrets.DOCKER_PASSWORD }}
          docker build -f Dockerfile -t ${{ secrets.DOCKER_REPO }}/kukathon .
          docker push ${{ secrets.DOCKER_REPO }}/kukathon

      - name: Deploy to Prod
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          envs: GITHUB_SHA
          script: |
            sudo docker-compose down
            sudo docker pull ${{ secrets.DOCKER_REPO }}/kukathon
            sudo nohup docker-compose up &

```