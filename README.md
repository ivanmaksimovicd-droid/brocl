using BepInEx;
using UnityEngine;
using UnityEngine.SceneManagement;

[BepInPlugin(
    "com.crossover.robloxbaldi",
    "Roblox Baldi Crossover",
    "0.1.0"
)]
public class RobloxBaldiMod : BaseUnityPlugin
{
    private GameObject noob;
    private Transform player;
    private float nextSearch;
    private bool spawned;

    private void Awake()
    {
        Logger.LogInfo("Roblox Baldi Crossover loaded!");
        SceneManager.sceneLoaded += OnSceneLoaded;
    }

    private void OnDestroy()
    {
        SceneManager.sceneLoaded -= OnSceneLoaded;
    }

    private void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        if (noob != null)
            Destroy(noob);

        noob = null;
        player = null;
        spawned = false;
        nextSearch = 0f;
    }

    private void Update()
    {
        if (player == null && Time.time >= nextSearch)
        {
            nextSearch = Time.time + 2f;
            FindPlayer();
        }

        if (player != null && noob == null)
            SpawnNoob();

        if (noob != null && player != null)
            ChasePlayer();
    }

    private void FindPlayer()
    {
        foreach (GameObject obj in
                 Resources.FindObjectsOfTypeAll<GameObject>())
        {
            if (!obj.scene.IsValid() || !obj.activeInHierarchy)
                continue;

            string n = obj.name.ToLowerInvariant();

            if (n == "player" || n == "playercontroller")
            {
                player = obj.transform;
                Logger.LogInfo("Found player: " + obj.name);
                return;
            }
        }

        Logger.LogDebug("Waiting to find the player object.");
    }

    private void SpawnNoob()
    {
        noob = new GameObject("RobloxNoob_Crossover");

        CreatePart("Head", new Vector3(0f, 1.65f, 0f),
            new Vector3(0.8f, 0.8f, 0.8f),
            new Color(1f, 0.8f, 0.15f));

        CreatePart("Torso", new Vector3(0f, 0.65f, 0f),
            new Vector3(0.9f, 1f, 0.45f),
            new Color(0.15f, 0.4f, 1f));

        CreatePart("LeftLeg", new Vector3(-0.23f, -0.2f, 0f),
            new Vector3(0.35f, 0.7f, 0.4f),
            new Color(0.15f, 0.8f, 0.2f));

        CreatePart("RightLeg", new Vector3(0.23f, -0.2f, 0f),
            new Vector3(0.35f, 0.7f, 0.4f),
            new Color(0.15f, 0.8f, 0.2f));

        noob.transform.position =
            player.position + player.forward * 5f;

        spawned = true;
        Logger.LogInfo("Roblox Noob prototype spawned.");
    }

    private void CreatePart(
        string partName,
        Vector3 localPosition,
        Vector3 scale,
        Color color)
    {
        GameObject part = GameObject.CreatePrimitive(
            PrimitiveType.Cube);

        part.name = partName;
        part.transform.SetParent(noob.transform, false);
        part.transform.localPosition = localPosition;
        part.transform.localScale = scale;

        // Visual prototype only: don't obstruct the original game.
        Collider collider = part.GetComponent<Collider>();
        if (collider != null)
            Destroy(collider);

        Renderer renderer = part.GetComponent<Renderer>();
        if (renderer != null)
        {
            renderer.material = new Material(
                Shader.Find("Diffuse") ??
                Shader.Find("Standard"));

            renderer.material.color = color;
        }
    }

    private void ChasePlayer()
    {
        Vector3 target = player.position;
        target.y = noob.transform.position.y;

        Vector3 direction = target - noob.transform.position;
        direction.y = 0f;

        if (direction.sqrMagnitude > 0.01f)
        {
            noob.transform.position +=
                direction.normalized * 2f * Time.deltaTime;

            noob.transform.rotation =
                Quaternion.LookRotation(direction);
        }
    }
}
