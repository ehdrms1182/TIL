

펄린노이즈 알고리즘
``` {.unity} using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class PulinNoise : MonoBehaviour
{
    [Header("[블록]")]
    public GameObject prefabBlock;


    [Header("[맵 정보]")]
    public int mapX = 0;
    public int mapY = 0;
    public float waveLength = 0;
    public float Amplitude = 0;

    //[Header("난수")]
    //int rand = Random.Range(1, 10);

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
}
```
