내가 쓰거나 유용했던 코드들
====

``` {.unity} using System.Collections;
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
    void Start()
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
        if (!test)
            return;
        for (int i = 0; i < BlockList.Count; i++)
        {
            float xCoord = (BlockList[i].transform.position.x) / waveLength;
            float zCoord = (BlockList[i].transform.position.z) / waveLength;
            int y = (int)(Mathf.PerlinNoise(xCoord, zCoord) * Amplitude);

            BlockList[i].transform.position = new Vector3(BlockList[i].transform.position.x, y, BlockList[i].transform.position.z);

        }
    }

```{.unity}
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FollowCam : MonoBehaviour
{
    public Transform target;
    public float dist = 25.0f;
    public float height = 50.0f;
    public float dampRotate = 0.0f;

    private Transform camPos;

    // Use this for initialization
    void Start()
    {
        camPos = GetComponent<Transform>();
    }

    // Update is called once per frame
    void LateUpdate()
    {

        float currYAngle = Mathf.LerpAngle(camPos.eulerAngles.y, target.eulerAngles.y, dampRotate * Time.deltaTime);

        Quaternion rot = Quaternion.Euler(0, currYAngle, 0);

        camPos.position = target.position - (rot * Vector3.forward * dist) + (Vector3.up * height);
        camPos.LookAt(target);
    }
}
```


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

    void Update()
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