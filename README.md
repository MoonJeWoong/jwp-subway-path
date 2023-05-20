# jwp-subway-path

1. API 기능 요구사항 명세

- [x] 노선에 역 등록 API 신규 구현
    - [x] 해당 노선에 역을 등록한다.
        - [x] POST `/lines/{lineId}/sections`
            - 필수 정보 : 노선 id, 상행 방향역 id, 하행 방향역 id, 역 사이의 거리 정보
- [x] 노선에 역 제거 API 신규 구현
    - [x] 해당 노선에서 역을 제거한다.
        - [x] DELETE `/lines/{lineId}/sections?station-id={stationId}`
            - 필수 정보 : 노선 id, 삭제할 역의 id
- [x] 노선 조회 API 기능 구현
    - [x] GET `/line-stations/{lineId}`
    - [x] 노선에 포함된 역을 순서대로 보여주도록 응답을 개선한다.
- [x] 노선 목록 조회 API 기능 구현
    - [x] GET `/line-stations`
    - [x] 노선에 포함된 역을 순서대로 보여주도록 응답을 개선한다.
- [x] 경로 조회 API 구현
  - [x] GET `/path`
  - [x] 출발역과 도착역 사이 최단 거리 경로를 구하는 API를 구현한다.
  - [x] 최단 거리 경로와 총 거리 정보를 함께 응답한다.
  - [x] 한 노선에서 경로 찾기 뿐만 아니라 여러 노선의 환승도 고려한다.


2. 도메인 기능 명세

- 역들 (Stations)
    - [x] 여러 역(station)에 대한 정보를 갖는다.
    - [x] 입력된 역이 존재하는 역인지 확인할 수 있다.
    - [x] 새로운 역을 추가할 수 있다.
        - [x] 중복되는 역이 입력되면 예외처리한다.
    - [x] 역을 삭제할 수 있다.
        - [x] 존재하지 않는 역을 삭제하려고 하면 예외처리한다.
    - [x] 등록된 역을 조회할 수 있다.
        - [x] 존재하지 않는 역을 조회하면 예외처리 한다.

- 역(Station)
    - [x] 역은 id, 이름을 갖는다.
    - [x] 역 이름이 동일한지 확인할 수 있다.

- 노선 구간들 (LineSections)
    - [x] 역마다 연결된 구간 정보를 관리한다.
    - [x] 서로 다른 노선에 포함되는 구간들로 초기화를 시도하는 경우 예외처리한다.
    - [x] 새로운 구간을 생성한다.
        - [x] 노선에 처음 역이 추가되는 경우
            - [x] `상행 종점 - 하행 종점`으로 구간이 생성된다.
        - [x] 노선의 기존 역을 기준으로 추가되는 경우
            - [x] 상행 방향 역이 신규역인 경우
                - [x] 구간 사이에 역이 추가될 때
                    - [x] 하행 방향 역이 포함된 해당 노선의 구간을 찾아서 두 구간으로 나눠준다.
                        - [x] `신규역 - 기존 하행 방향 역`, `기존 상행 방향 역 - 신규역` 구간을 추가한다.
                        - [x] `기존 상행 방향 역 - 기존 하행 방향 역` 구간을 삭제한다.
                - [x] 종착 구간 마지막에 역이 추가될 때
                    - [x] 상행 방향 역이 포함된 해당 노선의 구간을 찾아서 두 구간으로 나눠준다.
                        - [x] `기존 상행 방향 역 - 신규역`, `신규역 - 기존 하행 방향 역` 구간을 추가한다.
                        - [x] `기존 상행 방향 역 - 기존 하행 방향 역` 구간을 삭제한다.
                - [x] `새로운 역과 기존 역 사이 거리 >= 구간의 기존 거리` 인 경우 예외처리한다.
            - [x] 하행 방향 역이 신규역인 경우
                - [x] 구간 사이에 역이 추가될 때
                    - [x] 기존 하행 방향 역이 포함된 노선 구간이 1개인지 확인해서 상행 종착 구간인지 확인한다.
                    - [x] 상행 종착 구간임이 확인되면 입력된 `신규역 - 기존 상행 종착 역` 구간을 추가한다.
                - [x] 종착 구간 마지막에 역이 추가될 때
                    - [x] 기존 상행 방향 역이 포함된 노선 구간이 1개인지 확인해서 하행 종착 구간인지 확인한다.
                    - [x] 하행 종착 구간임이 확인되면 입력된 `기존 하행 종착 역- 신규역` 구간을 추가한다.
                - [x] `새로운 역과 기존 역 사이 거리 >= 구간의 기존 거리` 인 경우 예외처리한다.
        - [x] 상행역과 하행역이 이미 노선에 등록되어 있는 경우 예외처리한다.
    - [x] 역을 포함하는 구간들을 조회한다.
    - [X] 기존 구간을 삭제한다.
        - [X] 노선에서 역이 제거될 때는 `상행 방향 역 - 제거할 역`과 `제거할 역 - 하행 방향 역` 두 구간이 `상행 방향 역 - 하행 방향 역`으로 합쳐진다.
            - [X] 기존 제거할 역이 포함된 구간들은 삭제한다.
            - [X] 제거되는 역과 상행 방향 역, 하행 방향 역 사이의 거리의 합이 합쳐진 구간에 저장되어야 한다.
        - [X] 노선에 역이 마지막 두 개만 남은 상황에서 역을 제거할 때는 남은 모든 역이 삭제된다.
        - [X] 노선에 등록되지 않은 역을 제거하려 하는 경우 예외처리한다.
    - [x] 노선을 구성하는 구간들을 반환한다.

- 구간(Section)
    - [x] 구간은 id, 상행 방향 역(Station) 정보, 하행 방향 역(Station) 정보, 역 사이 거리, 노선을 갖는다.
    - [x] 상행 방향 역과 하행 방향 역이 동일하면 예외처리한다.

- 거리(Distance)
    - [x] 거리 정보가 양수가 아닐 경우 예외처리한다.

- 노선들(Lines)
    - [x] 노선들을 갖는다.
    - [x] 노선 이름은 중복일 수 없다.
    - [x] 노선 색상은 중복일 수 없다.

- 노선(Line)
- [x] 노선은 id, 이름, 정거장들의 정보를 갖는다.

- 경로(Path)
  - [x] 경로는 여러개의 역들과 총 소요 거리로 구성된다.

- 경로 탐색기(PathFinder)
  - [x] 출발지와 목적지 역이 주어지면 최단 경로와 거리를 찾아서 반환한다.
  - [x] 출발지와 목적지 역이 동일한 역으로 주어지면 예외처리한다.

- 요금 정책(FarePolicy)
  - [x] 요금 계산 기능
    - [x] 기본 운임(10km)이내 : 1,250원
    - [x] 추가 운임 부과 규정
    - [x] 10~50km : 5km 까지마다 100원 추가
    - [x] 50km 초과 : 8km 까지마다 100원 추가
  - [x] 주어진 거리가 음수일 경우 예외처리한다.


### Todo
- 프로그래밍 요구사항
    - [x] 프로덕션과 테스트 DB 설정을 분리한다.
        - [x] 프로덕션 DB는 로컬에 저장될 수 있도록 설정한다.
        - [x] 테스트용 DB는 인메모리로 동작하도록 한다.

