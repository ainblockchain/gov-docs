# API 서버

## API

### registerUser
- Description: User ID 를 등록하면 유저 개인 스토리지를 생성한다.

#### POST 요청 Body 메시지
```
{
  userId: string  // ".", "#", "$", "[",  "]" 를 제외한 모든 문자열이 가능하다.
  storageGb: number // Storage 용량 (단위: GB)
  localSchoolId: number // 캠퍼스 번호
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### getAllUserStatus
- Description: 모든 유저의 상태를 전달하며, 유저의 상태로는 현재 실행 중인 컨테이너 정보들과 clusterId 별 사용시간이 있다.

#### POST 요청 Body 메시지
```
{
  localSchoolId?: number // 없으면 전체 정보를 다 가져옴.
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
  useContainerCnt: number // 총 요청한 컨테이너 수(전체)
  runningContainerCnt: number // Running 상태인 컨테이너 수(전체)
  result: {
    [userId]: {
      useContainerCnt: number // 총 요청한 컨테이너 수(유저)
      runningContainerCnt: number // Running 상태인 컨테이너 수(유저)
      running_container_list: {
        clusterId: ‘cpu' | 'gpu'
        containerId: string
        status: string
        pending: 대기중
        success: 실행 중
        createdAt: number
        startedAt?: number
        timerStartedAt?: number
        maxDuration?: number
        storagePath?: string
        imagePath: string
        nodeId?: number
        remainingTime: number // 분, maxDuration 이 없으면 -1
        endpoint:
        jupyter: string
        shell: string
      }[] // Array
      usage: {
        cpu: number // 분단위
        gpu: number // 분단위
      },
    },
  }
}
```

### getUserStatus
- Description: 특정 유저의 상태를 전달하며, 유저의 상태로는 현재 실행 중인 컨테이너 정보들과 clusterId 별 사용시간이 있다.

#### POST 요청 Body 메시지
```
{
  userId: string
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
  useContainerCnt: number // 총 요청한 컨테이너 수
  runningContainerCnt: number // Running 상태인 컨테이너 수
  running_container_list: {
    clusterId: ‘cpu' | 'gpu'
    containerId: string
    status: string
    pending: 대기중
    success: 실행 중
    createdAt: number
    startedAt?: number
    timerStartedAt?: number
    maxDuration?: number
    storagePath?: string
    imagePath: string
    nodeId?: number
    remainingTime: number // 분, maxDuration 이 없으면 -1
    endpoint:
    jupyter: string
    shell: string
  }[] // Array
  usage: {
    cpu: number // 분단위
    gpu: number // 분단위
  }
}
```

### deleteUser
- Description: 유저와 관련된 컨테이너와 스토리지를 모두 제거한다. 하지만 유저의 기록을 유지한다.

#### POST 요청 Body 메시지
```
{
  userId: string
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### runContainer
- Description:컨테이너를 생성하고, 컨테이너와 관련된 정보를 전달한다. 관련된 정보는 컨테이너의 엔드포인트와 해당 컨테이너의 ID이다. 해당 API에 대한 응답은 컨테이너 생성이 완료된 시점이 아니다. 그래서 컨테이너 생성 상태는 getContainerStatus<API> 를 통해 확인한다.

#### POST 요청 Body 메시지
```
{
  userId: string
  clusterId: ‘cpu’ | 'gpu'
  nodeId?: number // aiffel 수업 번호.
  dockerImagePath?: string // docker image path
  max_duration?: number // 분단위 ,  ex. 2시간 이상 사용시 종료.
  storagePath?: string // 데이터 셋 공유 스토리지에 공유하고 싶은 위치
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
  endpoint: {
    shell: string
    jupyter: string
  }
  containerId: string
}
```

### editContainer
- Description: 특정 컨테이너의 이미지를 변경한다.

#### POST 요청 Body 메시지
```
{
  userId: string
  containerId: string
  nodeId?: string
  imagePath?: string
  storagePath?: string
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### terminateContainer
- Description:  특정 컨테이너를 제거한다.

#### POST 요청 Body 메시지
```
{
  userId: string
  containerId: string
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### terminateAllContainer
- Description: 모든 컨테이너를 제거한다. 해당 API는 개발용임.

#### POST 요청 Body 메시지
```
{}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### getContainerStatus
- Description: 특정 컨테이너의 상태를 전달한다.

#### POST 요청 Body 메시지
```
{
  userId: string
  containerId: string
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
  statusContainer: ‘pending’ | ‘createContainer’ | ‘success’ | ‘failed’
}
```
- 'pending': 생성 대기 중
- 'createContainer': 생성 중
- 'success': 생성 완료
- 'failed': 생성 실패

### extendContainer
- Description: 컨테이너 maxDuration 증가 요청

#### POST 요청 Body 메시지
```
{
  containerId: string
  userId: string
  maxDuration: number
}
```

#### POST 응답 메시지
```
{
  statusCode: number
  errMessage?: string
}
```

### getContainerQuota
- Description: clusterId 에 해당하는 최대 컨테이너 개수

#### POST 요청 Body 메시지
```
{
  localSchoolId?: number
}
```

#### POST 응답 메시지
```
{
  maxQuota?: {
    cpu: number
    gpu: number
  }
  maxQuotaSchool?: {
    [localSchoolId: string]: {
      cpu: number
      gpu: number
    }
  }
  statusCode: number
  errMessage?: string
}
```
