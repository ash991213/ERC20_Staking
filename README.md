## Simple ERC20 Token Staking Contract

- 블록체인 상에서 ERC20 토큰을 예치하여 다른 ERC20 토큰을 보상으로 받는 스테이킹 컨트랙트입니다.

- 토큰을 예치할 때는 타입에 따라 30, 60, 90일동안 예치하여 각각 10%, 25%, 50% 수익률 만큼의 보상 토큰을 받습니다.

- 토큰을 예치할 때 최소 예치량 이상 최대 예치량 이하만큼 예치할 수 있습니다.

- 토큰을 예치할 때는 최대 예치량 이하에 한해 추가로 예치할 수 있습니다.

- 예치한 토큰에 대한 스테이킹 보상을 받을 때 스테이킹 보상을 최신화하고 마지막 업데이트 시간을 변경합니다.

## 구성 요소

### stakingToken

- 스테이킹 토큰(ERC20)입니다.

### stakingToken

- 스테이킹 토큰(ERC20)입니다.

### rewardToken

- 보상 토큰(ERC20)입니다.

### minStake / maxStake

- 각각 최소 스테이킹 금액 / 최대 스테이킹 금액입니다.

### StakingType

- 스테이킹 타입을 열거형으로 정의한 것입니다.

### StakingPeriod

- 예치일 기준을 나타내는 구조체 입니다. `rateOne`, `rateTwo`, `rateThree`는 각각 30일, 60일, 90일 입니다.

### ReturnRate

- 보상률을 나타내는 구조체 입니다. `rateOne`, `rateTwo`, `rateThree`는 각각 10%, 25%, 50% 입니다.

### StakingInfo

- 스테이킹 현황을 나타내는 구조체입니다. `deposited`는 예치 금액, `firstAt`은 최초 스테이킹 시간, `updateAt`은 최종 업데이트 시간, `rewards`는 미지급 스테이킹 보상입니다.

### hourInSeconds

- 시간을 초 단위로 계산하기 위한 상수입니다.

### stakers

- 각 스테이커의 스테이킹 현황을 저장하기 위한 매핑입니다.

## 주의 사항

- `nonReentrant`를 사용하여 재진입 공격을 방지합니다.

- `SafeMatch` 라이브러리를 사용하여 스테이킹 보상을 계산할 때 오버플로우/언더플로우를 예방합니다.

- 스테이킹 토큰과 보상 토큰은 `immutable` 변수로 값을 변경할 수 없습니다.

- 스테이커는 스테이킹을 하기 위해 최소 금액 이상의 토큰을 스테이킹 컨트랙트로 보내야 합니다. 또한, 최대 스테이킹 금액 이상, 최소 스테이킹 이하의 금액은 스테이킹 컨트랙트로 보낼 수 없습니다.

- 스테이커는 스테이킹을 할 때 타입을 지정해야 합니다. 스테이킹 타입에 따라 스테이킹 기간과 보상률이 달라집니다.

- 스테이커는 각 타입의 스테이킹을 시작한 최초 시간부터 30일, 60일, 90일간 스테이킹을 할 수 있습니다.

- 스테이커는 스테이킹을 할 때 매 스테이킹 타입마다 한 번씩 스테이킹을 해야 합니다. 예를 들어, 스테이커가 30일 스테이킹을 예치했다면, 60일 스테이킹을 하기 위해서는 새로운 스테이킹을 해야합니다.

- 스테이커는 스테이킹을 할 때 최대 스테이킹 금액 내에서 추가 스테이킹이 가능합니다. 또한, 스테이킹이 종료되었을 경우 미지급 보상을 모두 받은 후 다시 스테이킹을 시도하여 반복하여 스테이킹을 할 수 있습니다.

- 스테이커는 예치한 토큰을 일부만 회수할 수 있습니다. 일부 회수를 할 경우 현재까지의 보상을 최신화하고 앞으로의 보상은 회수하고 남은 토큰의 양에 비례하여 지급받습니다.

- 스테이커는 예치한 토큰에 대한 보상을 받을 때 각 타입의 스테이킹 기간을 초과하여 지급하지 않습니다. 예를 들어, 30일 스테이킹을 하였다면 40일이 지난후에 스테이킹 보상을 받아도 30일에 대한 스테이킹 보상만 지급됩니다.

## Staking process

1.스테이커는 스테이킹에 사용할 토큰을 스마트 컨트랙트로 위임(approve)합니다.

2.스테이커는 `stake` 함수를 호출하여 토큰을 스테이킹 합니다. 이때, 금액(_amount)로 스테이킹할 토큰 양을 지정하고, 스테이킹 타입(_type) 인자로 스테이킹 타입을 지정합니다. 또한 금액(_amount) 값이 최소 및 최대 스테이킹 금액 사이여야 합니다.

2-1.만약 기존에 진행중인 스테이킹이 있다면, 최대 스테이킹 금액 내에서 추가 스테이킹을 할 수 있습니다.

2-2.스테이킹을 완료하고 다시 스테이킹을 시도할 경우, 남아있는 미지급 보상이 있다면 되돌리고 보상을 모두 받았다면 스테이킹 내용을 초기화하여 다시 스테이킹을 진행합니다.

3.스테이커는 `claimReward` 함수를 호출하여 보상을 청구할 수 있습니다. 이때, 스테이킹 타입(_type)을 인자로 보상을 청구할 스테이킹 타입을 지정합니다. 보상은 한 번에 하나의 스테이킹 타입에서만 청구할 수 있습니다. 스테이킹 기간 (30일, 60일, 90일)이 지나지 않았더라도 보상을 청구하면 최초 스테이킹 등록 시점부터 현재 시점만큼의 보상을 제공합니다.

4.스테이커는 `unstake` 함수를 사용하여 스테이킹을 해제할 수 있습니다. 이때, 스테이킹 타입(_type)을 인자로 스테이킹을 해제 할 스테이킹 타입을 지정합니다. 스테이킹을 해제하고 스테이킹 토큰을 돌려받았다면 `claimReward` 함수를 호출하여 보상을 청구할 수 있습니다.