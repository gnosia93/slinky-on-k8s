Slinky 환경에서 장애가 발생하면, Slurm의 작업 관리 메커니즘과 Kubernetes의 자가 치유(Self-healing) 기능이 동시에 작동합니다. Slurm fault tolerance(SchedMD) 원칙에 따른 시나리오별 동작은 다음과 같습니다.

### 1. 작업 중인 파드(Pod) 또는 프로세스 장애 ###
* Slurm의 감지: srun 프로세스가 비정상 종료되면 Slurm은 해당 작업을 FAILED 상태로 처리하고 로그(%j.out)에 에러를 기록합니다.
* 자동 재시도: 만약 sbatch 스크립트에 --requeue 옵션을 넣었다면, Slurm은 작업을 대기열(Queue)로 되돌려 다시 실행을 시도합니다.
* K8s의 역할: 개별 프로세스 장애 시 K8s는 개입하지 않고 Slurm의 판단에 맡깁니다.

### 2. Kubernetes 노드(EC2) 장애 (Node Panic/Termination) ###
* 노드 상태 변화: 노드가 응답하지 않으면 Slurm sinfo에서 해당 노드 상태가 down 또는 drain으로 변합니다.
* Karpenter/MNG의 대응:
  * Managed Node Group(정적): AWS가 노드 상태 불량을 감지하고 새 인스턴스를 띄워 Desired: 2 수치를 맞춥니다.
  * Karpenter(동적): 장애 노드를 삭제하고, 대기 중인 Slurm Pod을 위해 새로운 노드를 즉시 프로비저닝합니다. Karpenter Node Termination(Karpenter.sh) 문서에 자세한 과정이 나와 있습니다.

### 3. 장애 복구 후 재시작 (Checkpoint) ###
* 중요: 노드가 새로 떠도 학습은 처음부터 다시 시작됩니다.
* 대응책: 이를 방지하려면 Llama 3 모델 학습 시 Checkpoint 기능을 반드시 활성화해야 합니다. PyTorch Distributed Checkpoint를 쓰면 노드 장애 후 재투입 시 마지막 저장 지점부터 이어서 학습할 수 있습니다.

### 4. Slurm 컨트롤러(Slinky) 장애 ###
* 고가용성(HA): Slinky는 보통 K8s 상에서 Deployment로 돌아가므로, 컨트롤러 파드가 죽으면 K8s가 즉시 다른 노드에서 살려냅니다. 이때 작업 데이터는 DB(보통 외부 RDS)에 저장되어 있어 상태가 유지됩니다.

#### 💡 실무 팁: ####
sinfo에서 특정 노드가 drain 상태로 멈춰 있다면, 관리자가 scontrol update nodename=[노드명] state=resume 명령어로 수동 복구해줘야 할 수도 있습니다.
지금 발생한 특정 에러 메시지가 있나요? 아니면 장애 테스트(Chaos Testing)를 준비 중이신가요? 구체적인 상황을 알려주시면 대응 명령어를 정리해 드릴게요.
