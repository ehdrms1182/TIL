내가 쓰거나 유용했던 코드들
====

# 펄린노이즈를 이용한 랜덤 맵 생성 알고리즘
```{.unity}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class PulinNoise : MonoBehaviour
{
    [Header("[블록]")]
    public GameObject prefabBlock; //종류 늘릴 수 있음


    [Header("[맵 정보]")]
    public int mapX = 0;
    public int mapY = 0;
    public float waveLength = 0;
    public float Amplitude = 0;

    private List<GameObject> BlockList = new List<GameObject>();
    void Awake()
    {
        for(int x = 0; x<mapX; x++)
        {
            for(int z = 0; z<mapY; z++)
            {
                BlockList.Add((GameObject)Instantiate(prefabBlock, new Vector3(x, 0, z), Quaternion.identity));
            }
        }
       
    }

    public bool test = false;
    void Update()
    {
        for (int i = 0; i < BlockList.Count; i++)
        {
            float xCoord = (BlockList[i].transform.position.x) / waveLength;
            float zCoord = (BlockList[i].transform.position.z) / waveLength;
            int y = (int)(Mathf.PerlinNoise(xCoord, zCoord) * Amplitude);

            BlockList[i].transform.position = new Vector3(BlockList[i].transform.position.x, y, BlockList[i].transform.position.z);

        }
    }
}
```

# 카메라가 플레어이를 따라오게 만드는 스크립트

```{.unity}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FollowCam : MonoBehaviour
{
    public Transform target;
    public float distance = 25.0f;
    public float height = 50.0f;
    public float dampRotate = 0.0f;

    private Transform camPos;

    void Awake()
    {
        camPos = GetComponent<Transform>();
    }

    // Update is called once per frame
    void LateUpdate()
    {

        float currentYAngle = Mathf.LerpAngle(camPos.eulerAngles.y, target.eulerAngles.y, dampRotate * Time.deltaTime);

        Quaternion rotate = Quaternion.Euler(0, currentYAngle, 0);

        camPos.position = target.position - (rotate * Vector3.forward * distance) + (Vector3.up * height);
        camPos.LookAt(target);
    }
}
```


# 마우스 우클릭으로 원하는 방향으로 이동하는 스크립트

```{.unity}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ClickMove : MonoBehaviour
{
    [SerializeField]
    private Camera cam;
    public static bool isMove;
    private Vector3 location;
    public Transform character; // 추가

    //private float speed = 1;

    int layerMask;

    private void Awake()
    {
        layerMask = 1 << LayerMask.NameToLayer("Ground");
        cam = Camera.main;
        SetLocation(character.transform.position);//시작 위치를 캐릭터 위치로 설정
    }

    void LateUpdate()
    {
        GetLocation();
        Move();
    }

    private void GetLocation()
    {
        if (Input.GetMouseButton(1))
        {
            RaycastHit hit;
            if (Physics.Raycast(cam.ScreenPointToRay(Input.mousePosition), out hit, Mathf.Infinity, layerMask))
            {
                SetLocation(hit.point);
                transform.LookAt(hit.point);//클릭 방향을 바라보게한다
            }
        }
    }

    private void SetLocation(Vector3 pos)
    {
        Debug.Log("이동지점 설정");
        location = pos; 
        isMove = true; 
    }
    
    private void Move()
    {
        if (isMove)
        {
            if (Vector3.Distance(location, transform.position) <= 0.1f)
            { 
                isMove = false;
                return; 
            }
            var dir = location - transform.position; //현재 방향을 설정

            character.transform.position = Vector3.MoveTowards(character.transform.position,location,Time.deltaTime * 10f);//목적지까지 일정 속도로 이동
            //transform.position += dir.normalized * Time.deltaTime * 5f; 
        }
    }
}
```

# 빈 게임 오브젝트를 Gizmos를 이용해 시각화하는 코드 
# Gizmos -> Scene 안에 있는 게임 오브젝트 관련 그래픽
```{.unity}
public class GizmosColor : MonoBehaviour
{
    public Color color = Color.green; //Gizmos 색
    public float radius = 10f; //구의 반지름

    // 사용시 함수 2개중 1개 주석처리하고 사용

    void OnDrawGizmos() // 씬 화면 상에서 항상 보이는 설정
    {
        // Gizmos 색 변경
        Gizmos.color = color;
        // 구 모양의 Gizmos 생성
        Gizmos.DrawSphere(transform.position, radius); //Gizmos 생성 위치와 반지름을 인자값으로 준다
    }

    void OnDrawGizmosSelected // 선택시에만 보이는 설정
    {
        // Gizmos 색 변경
        Gizmos.color = color;
        // 구 모양의 Gizmos 생성
        Gizmos.DrawSphere(transform.position, radius); //Gizmos 생성 위치와 반지름을 인자값으로 준다
    }
}
```


# 싱글톤 코드
```{.unity}
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

/// <summary>
/// 싱글턴 베이스 클래스
/// </summary>
/// <typeparam name="T">Singleton<T>를 상속받는 타입</typeparam>
public abstract class Singleton<T> : MonoBehaviour where T : Singleton<T>
{
    /// <summary>
    /// 싱글턴 인스턴스를 찾거나 생성하는 과정 중 다른 스레드에서 사용중인지 판단할 객체
    /// </summary>
    private static object syncObject = new object();

    /// <summary>
    /// T 타입 인스턴스 객체
    /// </summary>
    protected static T instance;

    /// <summary>
    /// 외부에서 인스턴스 객체에 접근하기 위한 프로퍼티
    /// </summary>
    public static T Instance
    {
        get
        {
            // 인스턴스가 없다면
            if (instance == null)
            {
                // lock을 통한 동기화 과정은 일반적으로 유니티의 싱글 스레드 환경에서는 상관없지만, 
                // 멀티 스레드 환경에서 싱글톤 초기화 과정을 스레드 세이프하게 하기 위함 
                // (이 말은 즉 초기화 과정만 안전하고 그 이후는 스레드 세이프하지 않다는 뜻)
                // 인스턴스를 찾거나 생성하는 과정을 진행 중일 때 다른 스레드에서
                // 동일한 로직에 접근할 수 없게 락을 건다 (사용 중이라면 끝날 때까지 대기하게 됌)
                lock (syncObject)
                {
                    // 인스턴스를 찾는다
                    instance = FindObjectOfType<T>();
                    // 검색했지만 인스턴스가 없다면
                    if (instance == null)
                    {
                        // 인스턴스를 새로 생성
                        GameObject obj = new GameObject();
                        obj.name = typeof(T).Name;
                        instance = obj.AddComponent<T>();
                    }
                }
            }
            // 위의 과정을 통해 인스턴스가 있다면 그대로 반환
            // 없다면 찾거나 새로 생성해서 반환
            return instance;
        }
    }

    protected virtual void Awake()
    {
        lock (syncObject)
        {
            // 모노를 상속받았으므로 모든 객체의 초기화가 끝났을 때
            // Awake가 자동적으로 호출되는데 이 때 검사를 통해 인스턴스가 없다면 미리 넣어둔다
            // 이러한 방식으로 Instance 프로퍼티 접근 시 불필요한 검색 과정을 최대한 생략한다
            if (instance == null)
            {
                instance = this as T;
            }
            // 만약 이 때 인스턴스가 존재한다면 객체가 초기화 되기전에 해당 인스턴스에 접근하였다는 것
            // 해당 클래스는 모노를 상속받았고 일반적으로 씬(하이어라키)에 올려서 사용함
            // 그럼 이 때 인스턴스가 존재한다는 것은 씬에 올려진 인스턴스가 초기화되기 전에
            // 타 클래스에서 싱글톤으로 해당 클래스에 존재하여 별도의 인스턴스가 추가적으로 생성되었다는 의미
            // 따라서, 추가로 생성된 인스턴스를 삭제하고 먼저 씬에 배치한 싱글톤이 작동할 수 있도록 한다
            else
            {
                Destroy(instance);
            }
        }
    }

    private void OnDestroy()
    {
        lock (syncObject)
        {
            if (instance != this)
                return;

            instance = null;
        }
    }
}
```


# 카메라 흔들림 코드
```{.unity}
public class CameraShake : Monobehaviour
{
        [SerializeField]
    float force = 0f;
    Vector3 offset = Vector3.zero;
    Quaternion originRotate;

    private void Awake()
    {
        originRotate = transform.rotation;
    }

    IEnumerator ShakeCoroutine()
    {
        Vector3 originEuler = transform.eulerAngles;
        while(true)
        {
            float rotateX = Random.RandomRange(-offset.x, offset.x);
            float rotateY = Random.RandomRange(-offset.y, offset.y);
            float rotateZ = Random.RandomRange(-offset.z, offset.z);

            Vector3 randomRotate = originEuler + new Vector3(rotateX, rotateY, rotateZ);
            Quaternion rotation = Quaternion.Euler(randomRotate);

            while (Quaternion.Angle(transform.rotation, rotation) > 0.1f)
            {
                transform.rotation = Quaternion.RotateTowards(transform.rotation, rotation, force * Time.deltaTime);
                yield return null;
            }
            yield return null;
        }
    }
    
    private IEnumerator Reset()
    {
        while (Quaternion.Angle(transform.rotation, originRotate) > 0f)
        {
            transform.rotation = Quaternion.RotateTowards(transform.rotation, originRotate, force * Time.deltaTime);
        }
        yield return null;
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.K))
        {
            StartCoroutine(ShakeCoroutine());
        }
    }
}


```
