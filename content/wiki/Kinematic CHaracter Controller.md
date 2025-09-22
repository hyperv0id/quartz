## Basic

一个脚本控制角色：
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/26/20230326153332-39b412.png)

一个脚本描述 角色行为（注入一个 KCMotor，自己写一个脚本实现 ICharacterController接口）
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/26/20230326153426-019ee5.png)

### 创建 `MyPlayer` 脚本控制角色

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using KinematicCharacterController;
using KinematicCharacterController.Examples;
using System.Linq;

namespace KinematicCharacterController.Walkthrough.PlayerCameraCharacterSetup
{
    public class MyPlayer : MonoBehaviour
    {
        public ExampleCharacterCamera OrbitCamera;
        public Transform CameraFollowPoint;
        public MyCharacterController Character;

        private Vector3 _lookInputVector = Vector3.zero;

        private void Start()
        {
            Cursor.lockState = CursorLockMode.Locked;

            // Tell camera to follow transform
            OrbitCamera.SetFollowTransform(CameraFollowPoint);

            // Ignore the character's collider(s) for camera obstruction checks
            OrbitCamera.IgnoredColliders = Character.GetComponentsInChildren<Collider>().ToList();
        }

        private void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                Cursor.lockState = CursorLockMode.Locked;
            }
        }

        private void LateUpdate()
        {
            HandleCameraInput();
        }

        /// <summary>
        /// 处理相机输入
        /// </summary>
        private void HandleCameraInput()
        {
            // 鼠标输入
            float mouseLookAxisUp = Input.GetAxisRaw("Mouse Y");
            float mouseLookAxisRight = Input.GetAxisRaw("Mouse X");
            _lookInputVector = new Vector3(mouseLookAxisRight, mouseLookAxisUp, 0f);

            // 鼠标离开窗口
            if (Cursor.lockState != CursorLockMode.Locked)
            {
                _lookInputVector = Vector3.zero;
            }

            // 滚轮调整相机位置
            float scrollInput = -Input.GetAxis("Mouse ScrollWheel");

            // 将角色的转角 应用到 相机的转角
            OrbitCamera.UpdateWithInput(Time.deltaTime, scrollInput, _lookInputVector);

            // 右键切换相机 第一人称 和第三人称
            if (Input.GetMouseButtonDown(1))
            {
                OrbitCamera.TargetDistance = (OrbitCamera.TargetDistance == 0f) ? OrbitCamera.DefaultDistance : 0f;
            }
        }
    }
}
```

### 创建 `MyCharacterController` 脚本描述角色行为

实现 `ICharacterController` 接口，不实现任何方法，仅注入 Motor

```c#
public class MyCharacterController : MonoBehaviour, ICharacterController
{
	// KCC核心组件
	public KinematicCharacterMotor Motor;

	private void Start()
	{
		// 给 motor 注入，让其控制本角色
		Motor.CharacterController = this;
	}
	...
}
```

## 基础移动

### 设定输入

```c#
public struct PlayerCharacterInputs
{
	public float MoveAxisForward;
	public float MoveAxisRight;
	public Quaternion CameraRotation;
}   
```

玩家控制器 `MyPlayer` 会在每次 `Update()` 时处理 `WSAD` 等输入，并将其赋值给 `MyCharacterController`

```c#
// MyPlayer
private const string MouseXInput = "Mouse X";
private const string MouseYInput = "Mouse Y";
private const string MouseScrollInput = "Mouse ScrollWheel";
private const string HorizontalInput = "Horizontal";
private const string VerticalInput = "Vertical";

private void Update()
{
	if (Input.GetMouseButtonDown(0))
	{
		Cursor.lockState = CursorLockMode.Locked;
	}

	HandleCharacterInput();
}

private void HandleCharacterInput()
{
	PlayerCharacterInputs characterInputs = new PlayerCharacterInputs();
	
	characterInputs.MoveAxisForward = Input.GetAxisRaw(VerticalInput);
	characterInputs.MoveAxisRight = Input.GetAxisRaw(HorizontalInput);
	characterInputs.CameraRotation = OrbitCamera.Transform.rotation;

	// 将玩家的输入给到角色
	Character.SetInputs(ref characterInputs);
}
```

```c#
// MyCharacterController 接受输入，除了玩家输入，还可以使用程序模拟AI的输入
private Vector3 _moveInputVector;
private Vector3 _lookInputVector;
/// <summary>
/// This is called every frame by MyPlayer in order to tell the character what its inputs are
/// </summary>
public void SetInputs(ref PlayerCharacterInputs inputs)
{
	// Clamp input
	Vector3 moveInputVector = Vector3.ClampMagnitude(new Vector3(inputs.MoveAxisRight, 0f, inputs.MoveAxisForward), 1f);

	// Calculate camera direction and rotation on the character plane
	Vector3 cameraPlanarDirection = Vector3.ProjectOnPlane(inputs.CameraRotation * Vector3.forward, Motor.CharacterUp).normalized;
	if (cameraPlanarDirection.sqrMagnitude == 0f)
	{
		cameraPlanarDirection = Vector3.ProjectOnPlane(inputs.CameraRotation * Vector3.up, Motor.CharacterUp).normalized;
	}
	Quaternion cameraPlanarRotation = Quaternion.LookRotation(cameraPlanarDirection, Motor.CharacterUp);

	// Move and look inputs
	_moveInputVector = cameraPlanarRotation * moveInputVector;
	_lookInputVector = cameraPlanarDirection;
}
```

### 根据输入实现行为

相关变量

```c#
// 相关变量
[Header("Stable Movement")]
public float MaxStableMoveSpeed = 10f;
public float StableMovementSharpness = 15;
public float OrientationSharpness = 10;

[Header("Air Movement")]
public float MaxAirMoveSpeed = 10f;
public float AirAccelerationSpeed = 5f;
public float Drag = 0.1f;

[Header("Misc")]
public bool RotationObstruction;
public Vector3 Gravity = new Vector3(0, -30f, 0);
public Transform MeshRoot;
```

#### 转向行为

```c#
/// <summary>
/// Motor会在每次Update调用，只能在这里更新转向
/// </summary>
public void UpdateRotation(ref Quaternion currentRotation, float deltaTime)
{
	if (_lookInputVector != Vector3.zero && OrientationSharpness > 0f)
	{
		// Smoothly interpolate from current to target look direction
		Vector3 smoothedLookInputDirection = Vector3.Slerp(Motor.CharacterForward, _lookInputVector, 1 - Mathf.Exp(-OrientationSharpness * deltaTime)).normalized;

		// Set the current rotation (which will be used by the KinematicCharacterMotor)
		currentRotation = Quaternion.LookRotation(smoothedLookInputDirection, Motor.CharacterUp);
	}
}
```

#### 改变速度行为

```c#
/// <summary>
/// Motor会在每次Update调用，只能在这里更新速度
/// </summary>
public void UpdateVelocity(ref Vector3 currentVelocity, float deltaTime)
{
	Vector3 targetMovementVelocity = Vector3.zero;
	// 在地面上
	if (Motor.GroundingStatus.IsStableOnGround)
	{
		// 在这里平滑地计算目标速度，作者考虑了很多情况，所以更通用且复杂
		
		// Reorient source velocity on current ground slope (this is because we don't want our smoothing to cause any velocity losses in slope changes)
		currentVelocity = Motor.GetDirectionTangentToSurface(currentVelocity, Motor.GroundingStatus.GroundNormal) * currentVelocity.magnitude;

		// Calculate target velocity
		Vector3 inputRight = Vector3.Cross(_moveInputVector, Motor.CharacterUp);
		Vector3 reorientedInput = Vector3.Cross(Motor.GroundingStatus.GroundNormal, inputRight).normalized * _moveInputVector.magnitude;
		targetMovementVelocity = reorientedInput * MaxStableMoveSpeed;

		// Smooth movement Velocity
		currentVelocity = Vector3.Lerp(currentVelocity, targetMovementVelocity, 1 - Mathf.Exp(-StableMovementSharpness * deltaTime));
	}
	else
	{
		// Add move input
		if (_moveInputVector.sqrMagnitude > 0f)
		{
			targetMovementVelocity = _moveInputVector * MaxAirMoveSpeed;

			// Prevent climbing on un-stable slopes with air movement
			if (Motor.GroundingStatus.FoundAnyGround)
			{
				Vector3 perpenticularObstructionNormal = Vector3.Cross(Vector3.Cross(Motor.CharacterUp, Motor.GroundingStatus.GroundNormal), Motor.CharacterUp).normalized;
				targetMovementVelocity = Vector3.ProjectOnPlane(targetMovementVelocity, perpenticularObstructionNormal);
			}

			Vector3 velocityDiff = Vector3.ProjectOnPlane(targetMovementVelocity - currentVelocity, Gravity);
			currentVelocity += velocityDiff * AirAccelerationSpeed * deltaTime;
		}

		// 重力
		currentVelocity += Gravity * deltaTime;

		// Drag，可能是风阻
		currentVelocity *= (1f / (1f + (Drag * deltaTime)));
	}
}
```

### 完整代码

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using KinematicCharacterController;
using KinematicCharacterController.Examples;
using System.Linq;

namespace KinematicCharacterController.Walkthrough.BasicMovement
{
    public class MyPlayer : MonoBehaviour
    {
        public ExampleCharacterCamera OrbitCamera;
        public Transform CameraFollowPoint;
        public MyCharacterController Character;

        private const string MouseXInput = "Mouse X";
        private const string MouseYInput = "Mouse Y";
        private const string MouseScrollInput = "Mouse ScrollWheel";
        private const string HorizontalInput = "Horizontal";
        private const string VerticalInput = "Vertical";

        private void Start()
        {
            Cursor.lockState = CursorLockMode.Locked;

            // Tell camera to follow transform
            OrbitCamera.SetFollowTransform(CameraFollowPoint);

            // Ignore the character's collider(s) for camera obstruction checks
            OrbitCamera.IgnoredColliders.Clear();
            OrbitCamera.IgnoredColliders.AddRange(Character.GetComponentsInChildren<Collider>());
        }

        private void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                Cursor.lockState = CursorLockMode.Locked;
            }

            HandleCharacterInput();
        }

        private void LateUpdate()
        {
            HandleCameraInput();
        }

        private void HandleCameraInput()
        {
            // Create the look input vector for the camera
            float mouseLookAxisUp = Input.GetAxisRaw(MouseYInput);
            float mouseLookAxisRight = Input.GetAxisRaw(MouseXInput);
            Vector3 lookInputVector = new Vector3(mouseLookAxisRight, mouseLookAxisUp, 0f);

            // Prevent moving the camera while the cursor isn't locked
            if (Cursor.lockState != CursorLockMode.Locked)
            {
                lookInputVector = Vector3.zero;
            }

            // Input for zooming the camera (disabled in WebGL because it can cause problems)
            float scrollInput = -Input.GetAxis(MouseScrollInput);
#if UNITY_WEBGL
        scrollInput = 0f;
#endif

            // Apply inputs to the camera
            OrbitCamera.UpdateWithInput(Time.deltaTime, scrollInput, lookInputVector);

            // Handle toggling zoom level
            if (Input.GetMouseButtonDown(1))
            {
                OrbitCamera.TargetDistance = (OrbitCamera.TargetDistance == 0f) ? OrbitCamera.DefaultDistance : 0f;
            }
        }

        private void HandleCharacterInput()
        {
            PlayerCharacterInputs characterInputs = new PlayerCharacterInputs();

            // Build the CharacterInputs struct
            characterInputs.MoveAxisForward = Input.GetAxisRaw(VerticalInput);
            characterInputs.MoveAxisRight = Input.GetAxisRaw(HorizontalInput);
            characterInputs.CameraRotation = OrbitCamera.Transform.rotation;

            // Apply inputs to character
            Character.SetInputs(ref characterInputs);
        }
    }
}

```


```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using KinematicCharacterController;
using System;

namespace KinematicCharacterController.Walkthrough.BasicMovement
{
    public struct PlayerCharacterInputs
    {
        public float MoveAxisForward;
        public float MoveAxisRight;
        public Quaternion CameraRotation;
    }

    public class MyCharacterController : MonoBehaviour, ICharacterController
    {
        public KinematicCharacterMotor Motor;

        [Header("Stable Movement")]
        public float MaxStableMoveSpeed = 10f;
        public float StableMovementSharpness = 15;
        public float OrientationSharpness = 10;

        [Header("Air Movement")]
        public float MaxAirMoveSpeed = 10f;
        public float AirAccelerationSpeed = 5f;
        public float Drag = 0.1f;

        [Header("Misc")]
        public bool RotationObstruction;
        public Vector3 Gravity = new Vector3(0, -30f, 0);
        public Transform MeshRoot;

        private Vector3 _moveInputVector;
        private Vector3 _lookInputVector;
        
        private void Start()
        {
            // Assign to Motor
            Motor.CharacterController = this;
        }

        /// <summary>
        /// This is called every frame by MyPlayer in order to tell the character what its inputs are
        /// </summary>
        public void SetInputs(ref PlayerCharacterInputs inputs)
        {
            // Clamp input
            Vector3 moveInputVector = Vector3.ClampMagnitude(new Vector3(inputs.MoveAxisRight, 0f, inputs.MoveAxisForward), 1f);

            // Calculate camera direction and rotation on the character plane
            Vector3 cameraPlanarDirection = Vector3.ProjectOnPlane(inputs.CameraRotation * Vector3.forward, Motor.CharacterUp).normalized;
            if (cameraPlanarDirection.sqrMagnitude == 0f)
            {
                cameraPlanarDirection = Vector3.ProjectOnPlane(inputs.CameraRotation * Vector3.up, Motor.CharacterUp).normalized;
            }
            Quaternion cameraPlanarRotation = Quaternion.LookRotation(cameraPlanarDirection, Motor.CharacterUp);

            // Move and look inputs
            _moveInputVector = cameraPlanarRotation * moveInputVector;
            _lookInputVector = cameraPlanarDirection;
        }

        /// <summary>
        /// (Called by KinematicCharacterMotor during its update cycle)
        /// This is called before the character begins its movement update
        /// </summary>
        public void BeforeCharacterUpdate(float deltaTime)
        {
        }

        /// <summary>
        /// (Called by KinematicCharacterMotor during its update cycle)
        /// This is where you tell your character what its rotation should be right now. 
        /// This is the ONLY place where you should set the character's rotation
        /// </summary>
        public void UpdateRotation(ref Quaternion currentRotation, float deltaTime)
        {
            if (_lookInputVector != Vector3.zero && OrientationSharpness > 0f)
            {
                // Smoothly interpolate from current to target look direction
                Vector3 smoothedLookInputDirection = Vector3.Slerp(Motor.CharacterForward, _lookInputVector, 1 - Mathf.Exp(-OrientationSharpness * deltaTime)).normalized;

                // Set the current rotation (which will be used by the KinematicCharacterMotor)
                currentRotation = Quaternion.LookRotation(smoothedLookInputDirection, Motor.CharacterUp);
            }
        }

        /// <summary>
        /// (Called by KinematicCharacterMotor during its update cycle)
        /// This is where you tell your character what its velocity should be right now. 
        /// This is the ONLY place where you can set the character's velocity
        /// </summary>
        public void UpdateVelocity(ref Vector3 currentVelocity, float deltaTime)
        {
            Vector3 targetMovementVelocity = Vector3.zero;
            if (Motor.GroundingStatus.IsStableOnGround)
            {
                // Reorient source velocity on current ground slope (this is because we don't want our smoothing to cause any velocity losses in slope changes)
                currentVelocity = Motor.GetDirectionTangentToSurface(currentVelocity, Motor.GroundingStatus.GroundNormal) * currentVelocity.magnitude;

                // Calculate target velocity
                Vector3 inputRight = Vector3.Cross(_moveInputVector, Motor.CharacterUp);
                Vector3 reorientedInput = Vector3.Cross(Motor.GroundingStatus.GroundNormal, inputRight).normalized * _moveInputVector.magnitude;
                targetMovementVelocity = reorientedInput * MaxStableMoveSpeed;

                // Smooth movement Velocity
                currentVelocity = Vector3.Lerp(currentVelocity, targetMovementVelocity, 1 - Mathf.Exp(-StableMovementSharpness * deltaTime));
            }
            else
            {
                // Add move input
                if (_moveInputVector.sqrMagnitude > 0f)
                {
                    targetMovementVelocity = _moveInputVector * MaxAirMoveSpeed;

                    // Prevent climbing on un-stable slopes with air movement
                    if (Motor.GroundingStatus.FoundAnyGround)
                    {
                        Vector3 perpenticularObstructionNormal = Vector3.Cross(Vector3.Cross(Motor.CharacterUp, Motor.GroundingStatus.GroundNormal), Motor.CharacterUp).normalized;
                        targetMovementVelocity = Vector3.ProjectOnPlane(targetMovementVelocity, perpenticularObstructionNormal);
                    }

                    Vector3 velocityDiff = Vector3.ProjectOnPlane(targetMovementVelocity - currentVelocity, Gravity);
                    currentVelocity += velocityDiff * AirAccelerationSpeed * deltaTime;
                }

                // Gravity
                currentVelocity += Gravity * deltaTime;

                // Drag
                currentVelocity *= (1f / (1f + (Drag * deltaTime)));
            }
        }

        /// <summary>
        /// (Called by KinematicCharacterMotor during its update cycle)
        /// This is called after the character has finished its movement update
        /// </summary>
        public void AfterCharacterUpdate(float deltaTime)
        {
        }

        public bool IsColliderValidForCollisions(Collider coll)
        {

            return true;
        }

        public void OnGroundHit(Collider hitCollider, Vector3 hitNormal, Vector3 hitPoint, ref HitStabilityReport hitStabilityReport)
        {
        }

        public void OnMovementHit(Collider hitCollider, Vector3 hitNormal, Vector3 hitPoint, ref HitStabilityReport hitStabilityReport)
        {
        }

        public void PostGroundingUpdate(float deltaTime)
        {
        }

        public void AddVelocity(Vector3 velocity)
        {
        }

        public void ProcessHitStabilityReport(Collider hitCollider, Vector3 hitNormal, Vector3 hitPoint, Vector3 atCharacterPosition, Quaternion atCharacterRotation, ref HitStabilityReport hitStabilityReport)
        {
        }

        public void OnDiscreteCollisionDetected(Collider hitCollider)
        {
        }
    }
}
```


## 