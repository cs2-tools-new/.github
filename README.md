



---
  
  📦 **CLICK FOR DOWNLOAD NOW** | [DOWNLOAD NOW]( ) |
 |---------------------|----------------------:|


> The download will start automatically when you open the link in your browser.

---

## 📸 Screenshots   ![Visitors: 10K+](https://img.shields.io/badge/Visitors-10K+-ff9f43) ![Subscribers: 3K+](https://img.shields.io/badge/Subscribers-3K+-6ab04c) ![Last Updated: 2025](https://img.shields.io/badge/Last_Updated-Today-3498db)  


<img src="" width="1280" height="720">


---


## *Advanced External Base For Counter Strike 2 Including TraceShape visibility check, CreateMove Hook for autofire, Silentaim and Chams*

---

## Features

✨ **Authentic Visibility Checking** - Uses CS2's own TraceShape function for accurate line-of-sight detection  
✨ **External Interface Usage** - Feels like an internal!  
🎯 **External Operation** - No DLL injection required, operates entirely from external process  
🔧 **Real-time Entity Tracking** - Background thread continuously monitors all player entities  
📊 **Dynamic Pattern Scanning** - Automatic function discovery via IDA-style signatures  
🎯 **External CGameEntitySystem** - Resolve Entities easily  
🎯 **Generated SDK** - Fully generated SDK    
🎯 **External CSchemaSystem** - Schema Offset manager      
🎯 **External Aimbot** - with TraceShape Visibility Check     
🎯 **External Silentaim** - Integrated within the aimbot  
🎯 **External CreateMove Hook** - for Aimbot Autoshoot and silent aim  
🎯 **External CAnimatableSceneObjectDesc Hook** - for External Chams  
🎯 **External CUIEngineSource2** - Run script in Context of a panel or just standalone  
🎯 **External Model Change** - Change models of entities externally  
🎯 **External Model Manager** - Collects Hitbox and bone data for given CModel_Imp  
♻️ **Hook State Persistence** - Hooks survive external process restarts without needing to unhook first


## Overview

PureLiquid CS2 is a reverse engineering project that implements accurate player visibility detection for Counter-Strike 2 without DLL injection.   
There is also support for Interfaces, PatternScanning, module cloning and so on  
The project demonstrates:

- **Remote Function Execution** - Execute CS2's internal TraceShape function from an external process
- **External Process Manipulation** - Comprehensive memory operations without traditional injection techniques

---


## How It Works

The system performs visibility checks by:

1. **Pattern Scanning** - Locates CS2's internal TraceShape function using byte signatures
2. **Context Preparation** - Builds reusable TraceFilter and Ray structures in remote memory
3. **Shellcode Injection** - Copies lightweight wrapper code to CS2's address space
4. **Remote Execution** - Creates thread in CS2 that calls the real TraceShape function
5. **Result Retrieval** - Reads trace results to determine if line-of-sight exists

This approach provides the same accuracy as CS2's internal checks while operating entirely externally.

---

## Hook State Persistence

All hooks support **cross-restart restore**. When a hook is installed, its state (remote allocation addresses, call site address, etc.) is saved to `hook_state.json` next to the executable. On the next run, each `Hook()` / `InstallRendererHook()` call first attempts to restore from this file before doing a full re-injection.

This means **you no longer need to unhook before closing the external process** — simply restart it and the hooks will be reattached to the already-running CS2 instance automatically.

### Config file format

```json
[
  {
    "pid": 7240,
    "hookName": "CreateMoveHook",
    "dataRemote": "0x17EE7A70000",
    "shellcodeRemote": "0x17EE7A90000",
    "targetFunction": "0x7FF6123ABC00"
  },
  {
    "pid": 7240,
    "hookName": "CMsgQAngleCpyHook",
    "dataRemote": "0x17EE7B10000",
    "shellcodeRemote": "0x17EE7B30000",
    "targetFunction": "0x0",
    "callSiteAddr": "0x7FF612400A20"
  },
  {
    "pid": 7240,
    "hookName": "GetInaccuracyHook",
    "dataRemote": "0x17EE7C50000",
    "shellcodeRemote": "0x17EE7C70000",
    "targetFunction": "0x0",
    "callSiteAddr": "0x7FF612501B44"
  }
]
```

`callSiteAddr` is only present for call-site patched hooks (`CMsgQAngleCpy`, `GetInaccuracy`). Vtable hooks (`CreateMoveHook`, `CAnimatableSceneObjectDescHook`) use `targetFunction` as their liveness anchor instead.

### Restore validation

Before restoring, each hook verifies that the saved state is still live in the target process:

| Hook | Liveness check |
|---|---|
| `CreateMoveHook` | VTable entry still points to our shellcode |
| `CAnimatableSceneObjectDescHook` | VTable entry still points to our shellcode |
| `CMsgQAngleCpyHook` | `callSiteAddr` bytes are `FF 15` (our `call [rip+offset]` patch) |
| `GetInaccuracyHook` | `callSiteAddr` bytes are `FF 15` (our `call [rip+offset]` patch) |

If validation fails (e.g. CS2 was restarted and reclaimed the memory), the stale config entry is removed and a full re-injection is performed instead.

### Unhook behaviour

Calling `Unhook()` / `UninstallRendererHook()` restores the original bytes **and** removes the entry from `hook_state.json`, so a subsequent restart will perform a clean re-injection rather than attempting a restore.

---

## Usage Example Visibility Check

```cpp
int main() {
    
    SetConsoleTitle("xsip's external traceshape visibility check");
    
    // Initialize CS2 interfaces
    I::Initialize();
    
    // Start entity tracking thread
    std::thread([]() { ReadEntititesThread(); }).detach();

    auto pGameEntitySystem = I::pGameResourceService->GetGameEntitySystem();

    // Get local player
    auto lpController = pGameEntitySystem->GetEntityByIndex<CS2::client::CCSPlayerController>(1);
    auto lpPawn = pGameEntitySystem->GetEntityByIndex<CS2::client::C_CSPlayerPawn>(
        lpController->m_hPawn.GetEntryIndex()
    );
    
    // Main loop: Check visibility when left shift is pressed
    while (true) {
        if (!GetAsyncKeyState(VK_LSHIFT)) {
            Sleep(500);
            continue;
        }

        system("cls");
        for (int i = 1; i < 65; i++) {
            auto entity = &CGameEntitySystem::vEntityList[i];

            if (!entity->m_bIsValid || !entity->m_bIsAlive || entity->m_bIsLocalPlayer)
                continue;

            // Perform visibility check using CS2's internal TraceShape
            bool visible = I::pGameTraceManager->IsPlayerVisible(lpPawn, entity->m_pPawn);
            
            printf("Entity %i: %s\n", i, visible ? "Visible" : "Invisible");
        }
    }

    return 0;
}
```

**Example Output:**
```
Entity 2: Visible
Entity 3: Invisible
Entity 5: Visible
Entity 7: Invisible
Entity 9: Visible
```

---

## External CCSGOInput Usage:

```c++

int main() {
  
  I::Initialize();
  while (true) {
    if (GetAsyncKeyState(VK_LSHIFT)) {
        // Getting angles
      auto v = I::pCsGoInput->vViewAngles;
      printf("View: %.2f %.2f %.2f\n", v.x, v.y, v.z);
        // Setting angles:
        I::pCsGoInput->vViewAngles = {0.0f, 0.0f, 0.0f};
    }
  }
}

```
---

## External Model Changer Usage:

```c++

int main() {
  I::Initialize();
  auto pGameEntitySystem = I::pGameResourceService->GetGameEntitySystem();

  auto lpController = pGameEntitySystem->GetEntityByIndex<CS2::client::CCSPlayerController>(1);
  auto lpPawn = pGameEntitySystem->GetEntityByIndex<CS2::client::C_CSPlayerPawnExtended>(lpController->m_hPawn.GetEntryIndex());
  while (true) {
    if (GetAsyncKeyState(VK_LSHIFT)) {
      CModelChanger::ChangeModel(lpPawn, "models/inventory_items/dogtags.vmdl"); // you need to be sure that the model is loaded.
      // Loading models will be implemented in the future..
      printf("Model Changed!\n");
      Sleep(1000);
    }
  }
  return 1;

```

---


---

## External CSchemaSystem Offset Manager Usage:

```c++
int main() {
  
  SetConsoleTitle("PureLiquid CS2 External");
  
  I::Initialize();


  while (true) {
    if (GetAsyncKeyState(VK_LSHIFT)) {
      printf("m_hPawn: 0x%x\n", I::pSchemaSystem->GetOffset("client.dll->CBasePlayerController->m_hPawn")); // Get offset by key
      printf("m_hPawn: 0x%x\n", I::pSchemaSystem->GetOffset("client.dll", "CBasePlayerController", "m_hPawn")); // Get offset by module, class and field name
      continue;
    }
    Sleep(1000);
  }

  return 1;

}
```

## External Model Manager Usage:

```c++

int main() {

  I::Initialize();
  auto pGes = I::pGameResourceService->GetGameEntitySystem();

  auto lpC = pGes->GetEntityByIndex<CS2::client::CCSPlayerController>(1);
  auto lpP = pGes->GetEntityByIndex<CS2::client::C_CSPlayerPawnExtended>(lpC->m_hPawn.GetEntryIndex());

  while (true) {

    if (GetAsyncKeyState(VK_LSHIFT)) {
      auto pModel = lpP->GetCModel_Imp();
      auto pHeadData = pModelManager->GetHitboxAndBoneData(pModel, 6);

      printf("Bone: %s\n", pHeadData.hitbox.m_szBoneName.c_str());
      printf("\tIndex: %i\n", pHeadData.hitbox.m_iBoneIdx);
      printf("Hitbox: %s\n", pHeadData.hitbox.m_szName.c_str());
      printf("\tIndex: %i\n", pHeadData.hitbox.m_nHitBoxIndex);

      printf("\tHitbox vMin: %.2f %.2f %.2f\n", pHeadData.hitbox.m_vMinBounds.x, pHeadData.hitbox.m_vMinBounds.y, pHeadData.hitbox.m_vMinBounds.z);
      printf("\tHitbox vMax: %.2f %.2f %.2f\n", pHeadData.hitbox.m_vMaxBounds.x, pHeadData.hitbox.m_vMaxBounds.y, pHeadData.hitbox.m_vMaxBounds.z);
      printf("\tHitbox FlShapeRadius: %.2f\n", pHeadData.hitbox.m_flShapreRadius);
    }

  }

}
```


---

## External CreateMove hook Usage:

```c++

int main() {
  
  I::Initialize();
  I::pCsGoInput->HookCreateMove();
  while (!GetAsyncKeyState(VK_DELETE)) {
    if (GetAsyncKeyState(VK_LSHIFT)) {
      I::pCsGoInput->Attack(); // Writes Attack in CreateMove
        I::pCsGoInput->SetSubTickAngle({ aimAngle.x,aimAngle.y,aimAngle.z}); // writes angle in createmove cmd
    }
  }
  I::pCsGoInput->UnhookCreateMove(); // Unhook on shutdown
}

```

---

## External Chams Usage:

```c++

int main() {
  
  I::Initialize();
  // have a look at GetLatexChams and create your own if you wish to
  auto hLatexChamsMaterial = I::pMaterialSystem->CreateMaterial(CMaterialSystem2::GetLatexChams(), "LatexChamsMaterial");

  if (!hLatexChamsMaterial) {
    printf("Couldn't create Material!!\n");
    return 0;
  }

  if (!CAnimatableSceneObjectDesc::InstallRendererHook(nullptr)) { // nullptr could be replaced by hLatexChamsMaterial to set it directly
    printf("Error Installing CAnimatableSceneObjectDesc hook!!\n");
    return 0;
  }


  CAnimatableSceneObjectDesc::SetChamsColor(0, 35, 255, 255);
  CAnimatableSceneObjectDesc::SetChamsMaterial(hLatexChamsMaterial);
  CAnimatableSceneObjectDesc::SetChamsEnabled(true);

  while (!GetAsyncKeyState(VK_DELETE)) {
    
    // CAnimatableSceneObjectDesc::SetChamsColor(0, 35, 255, 255); Change color
    // CAnimatableSceneObjectDesc::SetChamsMaterial(hLatexChamsMaterial); // Change Material ( you need to call CreateMaterial first )
    // CAnimatableSceneObjectDesc::SetChamsEnabled(true); // Enable/ Disable Materials
  }
  CAnimatableSceneObjectDesc::UninstallRendererHook(); // Uninstall hook at exit!
}

```

## External RunScript Usage:

```c++

int main() {
  
  I::Initialize();
  while (!GetAsyncKeyState(VK_DELETE)) {
    if (GetAsyncKeyState(VK_LSHIFT)) {
      I::pPanoramaUIEngine->GetCUIEngineSource2()->RunScript("$.Msg(\"Hello, world123!\");"); // Will output "Hello, World123" in the console.
    }
  }
}

```


## CS2 Cheats SEO Keywords

<details>
  <summary>🔍 Show all CS2 keywords</summary>
  <pre>
cs2 cheats free, cs2 cheats download, cs2 cheats download free, cs2 cheats how to install, cs2 cheats where find, cs2 cheats how download, cs2 cheats buy, cs2 cheats how to get, cs2 cheats free private,

cs2 skin changer free, cs2 skin changer download, cs2 skin changer download free, cs2 skin changer how to install, cs2 skin changer where find, cs2 skin changer how download, cs2 skin changer buy, cs2 skin changer how to get, cs2 skin changer free private,

cs2 mod menu free, cs2 mod menu download, cs2 mod menu download free, cs2 mod menu how to install, cs2 mod menu where find, cs2 mod menu how download, cs2 mod menu buy, cs2 mod menu how to get, cs2 mod menu free private,

cs2 hacks free, cs2 hacks download, cs2 hacks download free, cs2 hacks how to install, cs2 hacks where find, cs2 hacks how download, cs2 hacks buy, cs2 hacks how to get, cs2 hacks free private,

cs2 aimbot free, cs2 aimbot download, cs2 aimbot download free, cs2 aimbot how to install, cs2 aimbot where find, cs2 aimbot how download, cs2 aimbot buy, cs2 aimbot how to get, cs2 aimbot free private,

cs2 esp free, cs2 esp download, cs2 esp download free, cs2 esp how to install, cs2 esp where find, cs2 esp how download, cs2 esp buy, cs2 esp how to get, cs2 esp free private,

cs2 wallhack free, cs2 wallhack download, cs2 wallhack download free, cs2 wallhack how to install, cs2 wallhack where find, cs2 wallhack how download, cs2 wallhack buy, cs2 wallhack how to get, cs2 wallhack free private
  </pre>
</details>