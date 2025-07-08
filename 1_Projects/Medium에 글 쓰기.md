## 도입
인포크스토어 그리고 인포크링크로 시작한 ABZ는 그 규모가 성장함에 따라 다양한 서비스를 출시하게 되었습니다. 이 과정에서 늘어가는 레포지토리를 효과적으로 관리하기 위해 최근 모노레포를 도입했습니다.

기존의 서비스들은 차례차례 migration을 진행 중이며, 최근 신규 출시한 인포크매니저를 비롯해 다양한 서비스를 모노레포 내에서 개발하고 있는 중인데요,
이번 글에서는 모노레포 환경 내에서 next.js 프로젝트를 신규로 개발하며 배운 점과, docker를 활용해 배포하는 과정에 대해 소개합니다.

> 이 글을 통해 다음 내용에 대해 알 수 있어요.
> next.js의  standalone 옵션
> next.js에서 env 파일의 적용 과정
> standalone 옵션과 docker를 활용해 경량화된 next.js 이미지 만들기

## 내용
### 1. next.js standalone
모노레포의 구축에 있어 turbo를 활용했습니다. 모노레포 환경에서 docker를 활용하는 것은 처음이었기에 가장 먼저 [turbo 공식문서](https://turbo.build/repo/docs/guides/tools/docker)를 읽으며 docker를 같이 활용하는 방식에 대해 학습했습니다. 이 과정에서 아래와 같은 문구를 읽게 되었습니다.

```
This Dockerfile is written for a Next.js app that is using the standalone output mode.
```

도대체 어떤 옵션이기에 turbo 문서에서도 이렇게 소개를 하고 있는 건지 궁금해져서 간단하게 아래와 같이 정리해봤습니다.

[공식 문서](https://nextjs.org/docs/pages/api-reference/next-config-js/output)의 설명에 따르면, Next.js는 빌드 과정에서 각 페이지와 의존성들을 훑으며 production 버전에 필요한 파일들을 결정합니다.
만약 이 과정에서 불필요한 파일이 빌드에 포함된다면, 결과물의 용량 또한 커지게 될 것이고 이로 인해 배포 시간, 트래픽 비용 등에 있어 문제가 발생할 수 있게 됩니다.

`standalone` 옵션은 이름 그대로(독립적, 독립형) 추가적인 종속성 설치 없이 단독으로 애플리케이션이 실행되게 합니다. `.next/standalone` 폴더 내에 최소한의 종속성과 서버를 구동시키기 위한 `server.js` 파일이 포함되기 때문입니다.

![[Pasted image 20241105011749.png]]

이렇게 구동되는 최소한의 서버는 CDN의 사용을 권장하는 `public`, `.next/static` 디렉토리를 제외하고 production 배포에 꼭 필요한 파일들만 포함시킵니다. 물론 해당 디렉토리의 파일이 필요하다면 수동으로 복사할 수 있습니다.

### 2. docker를 활용해 next.js 이미지 만들기
standalone 옵션에 대해 알았으니 이제 더 가벼운 docker 이미지를 만들 차례입니다.
쉽고 빠른 이해를 위해 먼저 docker의 주요 개념에 대해 간단하게 알아볼게요.

**컨테이너**
- docker에서 가장 핵심이 되는 개념
- 프로그램과 그 프로그램이 필요한 모든 설정이나 파일을 하나의 상자처럼 묶은 것

**이미지**
- 컨테이너를 만들기 위해 필요한 설계도같은 개념
- 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것

**Dockerfile**
- 이미지를 만들기 위한 설정 파일

**Dockerfile 작성 문법**

| 명령어        | 설명                                                                                                                        |
| ---------- | ------------------------------------------------------------------------------------------------------------------------- |
| FROM       | base 이미지 설정                                                                                                               |
| WORKDIR    | - 작업 디렉토리를 지정한다. 해당 디렉토리가 없으면 새로 생성<br>- 작업 디렉토리를 지정하면 그 이후 명령어는 해당 디렉토리를 기준으로 동작한다.                                      |
| RUN<br>    | - 새로운 레이어에서 명령어를 실행하고, 새로운 이미지를 생성한다.<br>- RUN 명령어 실행시마다 레이어가 생성되고 캐시된다.<br>- 이미지를 빌드할 때 실행                               |
| CMD        | - 컨테이너가 시작될 때 실행할 커맨드를 지정                                                                                                 |
| ENTRYPOINT | - 컨테이너가 시작될 때 실행될 커맨드                                                                                                     |
| EXPOSE     | - 컨테이너가 실행될 때 컨테이너로 들어오는 트래픽을 특정 포트(port)로 받아들일 수 있도록(listen) 지정하는 역할<br>- 컨테이너 생성 시 -p 옵션의 컨테이너 포트 값으로 EXPOSE 값을 적어야 한다. |
| COPY       | - host 내에 있는 파일 또는 디렉토리를 컨테이너의 파일 시스템으로 복사<br>- --chown 옵션으로 파일과 디렉토리에 대한 소유 권한을 지정할 수 있다                                 |
| ADD        | - 파일 또는 디렉토리를 컨테이너로 복사<br>- COPY와 거의 동일하나, ADD는 host 내에 있는 파일 외에도 경로를 지정해 remote 파일/디렉토리를 복사해올 수 있다                       |
| ENV        | - 컨테이너 실행 시점에 사용할 환경 변수를 설정하는 키워드                                                                                         |
| ARG        | - 빌드 시점에만 사용할 수 있는 변수<br>- ARG로 설정된 값은 이미지가 빌드된 후에는 유지되지 않으며, 컨테이너 실행 시점에서는 사용할 수 없다                                      |
| USER       | - 컨테이너 안에서 명령을 실행할 유저명, 유저그룹을 지정<br>- 기본적으로 컨테이너는 root 계정으로 실행되는데 이를 사용자계정 등으로 변경하기 위함                                    |

본격적인 Dockerfile 작성에 앞서 프로젝트의 구조를 간단하게 설명하면 다음과 같습니다.
- pnpm, turbo를 활용한 모노레포
- services 디렉토리 내에 다양한 서비스 존재

이제 완성된 Dockerfile을 보여드린 뒤 각 단계에 대해 차례차례 설명할게요.
```Dockerfile
# 1. 베이스 이미지 설정 (node와 Alpine 기반)
FROM node:18-alpine AS base
ENV PNPM_HOME="/pnpm"
ENV PATH="${PNPM_HOME}:${PATH}"

# 필요한 라이브러리 설치 (glibc 호환 포함)
RUN apk update && apk add --no-cache libc6-compat git openssh

# SSH key 설정
ARG SSH_PRIVATE_KEY
RUN mkdir ~/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" | tee ~/.ssh/id_rsa
RUN chmod 600 ~/.ssh/id_rsa

# private repository 설치를 위한 Github Authorization
RUN ssh-keyscan github.com >> ~/.ssh/known_hosts

# 2. 빌드 단계 설정
FROM base AS builder

ARG NODE_ENV
ENV NODE_ENV=$NODE_ENV
ARG STAGE
ENV STAGE=$STAGE

# 작업 디렉토리 설정
WORKDIR /app

# 전체 코드 복사 후 pruned 작업
COPY . .

# STAGE 값에 따라 .env 파일을 다르게 복사
RUN if [ "$STAGE" = "development" ]; then \
		echo "Using development environment" && \
		cp services/프로젝트명/.env.development services/프로젝트명/.env.local; \
	elif [ "$STAGE" = "production" ]; then \
		echo "Using production environment" && \
		cp services/프로젝트명/.env.production services/프로젝트명/.env.local; \
	else \
		echo "Invalid STAGE value: $STAGE" && exit 1; \
	fi

# pnpm과 turbo 설치
RUN npm install -g pnpm turbo

# 모노레포의 루트 package.json과 lock 파일 복사
COPY package.json pnpm-lock.yaml ./

# 의존성 설치 (루트 의존성 포함)
RUN pnpm install --frozen-lockfile

# 프로젝트에 필요한 의존성만 남기고 prune 수행
RUN turbo prune 프로젝트명 --docker

# 3. 의존성 설치 단계
FROM base AS installer

ARG NODE_ENV
ENV NODE_ENV=$NODE_ENV
ARG STAGE
ENV STAGE=$STAGE

# 작업 디렉토리 설정
WORKDIR /app

# pruned 의존성 복사 및 설치
COPY --from=builder /app/out/json/ .
RUN npm install -g pnpm turbo
RUN pnpm install --frozen-lockfile

# 프로젝트 전체 복사 및 빌드
COPY --from=builder /app/out/full/ .
RUN pnpm turbo run build --filter=프로젝트명...

# 4. 최종 런타임 단계 설정
FROM base AS runner

ARG NODE_ENV
ENV NODE_ENV=$NODE_ENV
ARG STAGE
ENV STAGE=$STAGE

# 작업 디렉토리 설정
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 프로젝트 실행에 필요한 파일들 복사
COPY --from=installer --chown=nextjs:nodejs /app/services/프로젝트명/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/services/프로젝트명/.next/static ./services/프로젝트명/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/services/프로젝트명/public ./services/프로젝트명/public

# 애플리케이션 실행
EXPOSE 80
ENV PORT=80

CMD ["node", "services/프로젝트명/server.js"]
```

**1. base 단계**
> 이미지 기반 설정과 SSH 키를 통한 접근 설정

경량의 배포를 위해 Alpine을 활용해 이미지 기반 설정을 진행합니다.
프로젝트에서 활용하는 디자인 시스템의 설치를 위해 SSH 키를 활용해 접근 권한을 부여합니다.

**2. builder 단계**
> 환경 변수에 맞춰 설정 파일을 복사하고 turbo prune으로 최적화

해당 단계에서는 STAGE 변수를 활용해 실행하고자 하는 환경을 정합니다.
next.js에서는  `.env.$(NODE_ENV).local` > `.env.local` > `.env.${NODE_ENV}` 순서로 env 변수들이 적용되기 때문에, 원하는 env 파일을 `env.local` 파일로 복사합니다 (출처 : [Next.js 공식문서](https://nextjs.org/docs/pages/building-your-application/configuring/environment-variables#environment-variable-load-order))

`turbo prune 프로젝트명` 명령어는 해당 프로젝트에 꼭 필요한 의존성만 남기고 불필요한 부분을 제거하여 최적화합니다. (출처 : [turbo 공식문서](https://turbo.build/repo/docs/guides/tools/docker#the-solution))

**3. installer 단계**
> 최적화된 의존성 설치 및 프로젝트 빌드

해당 단계에서는 builder 단계에서 최적화된 의존성 파일을 복사하고 설치하는 작업을 진행합니다.

**4. runner 단계**
> 최소한의 파일만 복사해 애플리케이션 실행

마지막으로 최종 실행 환경을 설정하며, 최적화된 파일을 복사합니다.
위에서 설명했듯 standalone 옵션은 `.next/static/`, `public` 디렉토리를 포함시키지 않기 때문에 수동으로 복사하는 작업을 진행합니다.

마지막으로 80번 포트를 열어 트래픽을 받을 수 있게 합니다.


그렇다면 standalone 옵션을 적용해 빌드된 next.js 이미지의 용량은, 옵션을 적용하지 않았을 때와 얼마나 차이가 있는지 확인해볼까요?


이렇게 만들어진 이미지를 활용해 더 빠르고 가벼운 next.js 프로젝트를 배포할 수 있습니다.

## 결론 및 인사
처음 접해보는 옵션이라 생소했지만 직접 docker를 활용하며 비교하다 보니 그 내용을 더 잘 이해할 수 있었던 것 같아요. 프론트엔드 개발자로서 UI, 컴포넌트 개발만 하는 데서 그치지 않고 서비스 전체에 걸쳐 학습하며 한 걸음 더 성장할 수 있었습니다.
현재 next.js로 개발되어 있는 다른 프로젝트들의 경우에도 시간이 되면 standalone 옵션을 차차 검토 후 적용해볼 예정입니다.

ABZ에는 정해진 업무만 하는 것이 아닌, 끊임없이 고민하고 우리의 힘을 활용해소비자를 만족시키고자 노력하는 엔지니어들이 가득합니다. 누구보다 빠르게 성장할 수 있는 ABZ에 대해 관심이 있으시다면 여기를 눌러주세요.

FE engineer 김학재