# starrail API Documentation (Expanded)

starrail is a Node.js/TypeScript library for extracting, manipulating, and analyzing Honkai: Star Rail game data. It provides a high-level API for accessing character, light cone, relic, and user data, as well as utilities for cache management and localization.

---

## Main Classes & Interfaces

### StarRail
- **Purpose:** Main client for loading and managing game data, cache, and user info.
- **Constructor:**
  ```js
  const client = new StarRail({
    defaultLanguage: 'en',
    cacheDirectory: './.cache',
    showFetchCacheLog: true
  });
  ```
- **Key Methods:**
  - `fetchUser(uid)`: Fetches player data by UID. Returns a `StarRailUser`.
  - `getAllCharacters()`: Returns all available `CharacterData` objects.
  - `getAllLightCones()`: Returns all available `LightConeData` objects.
  - `cachedAssetsManager`: Manages cache and raw game data.
  - `getExcelData(type)`: Accesses raw data tables (see `ExcelType`).

### CachedAssetsManager
- **Purpose:** Handles cache, updates, and access to raw game data (characters, light cones, relics, etc.).
- **Key Methods:**
  - `fetchAllContents()`: Updates all cache data
  - `getExcelData(type)`: Accesses raw data tables
  - `cacheDirectoryPath`: Path to cache

### CharacterData
- **Purpose:** Static info for a character (name, path, element, skills, traces, eidolons, icons)
- **Key Properties:**
  - `id`, `name`, `description`, `stars`, `maxEnergy`, `combatType`, `path`, `skills`, `skillTreeNodes`, `eidolons`, `icon`, `sideIcon`, etc.
- **Key Methods:**
  - `getStatsByLevel(ascension, level)`: Returns stats for a given ascension/level
  - `getGroupedSkillTreeNodes()`: Returns grouped trace nodes

### Character
- **Purpose:** Represents a character instance with stats, skills, traces, and eidolons.
- **Builder Pattern:**
  ```js
  const character = Character.builder()
    .character(characterData, true)
    .level(80)
    .ascension(6)
    .eidolons(0)
    .build(client);
  ```
- **Key Properties:**
  - `characterData`: Static data (see above)
  - `skills`: Array of `LeveledSkill` objects
  - `skillTreeNodes`: Array of `LeveledSkillTreeNode` (traces)
  - `eidolons`: Array of `Eidolon` objects
  - `stats`: Calculated stats at current level/ascension
  - `basicStats`: Array of `StatPropertyValue`

### Skill, LeveledSkill, SkillLevel
- **Skill:** Static skill definition (id, type, icon, etc.)
- **LeveledSkill:** Skill at a specific level, with parameters and description.
- **SkillLevel:** Represents a skill's level (value, extra)
- **Key Methods:**
  - `getSkillByLevel(level: SkillLevel): LeveledSkill`

### SkillTreeNode, LeveledSkillTreeNode
- **SkillTreeNode:** Represents a trace node (major/minor) in the skill tree.
- **LeveledSkillTreeNode:** Node at a specific level, with stats and description.
- **Key Methods:**
  - `getSkillTreeNodeByLevel(level: SkillLevel): LeveledSkillTreeNode`
  - `getFullDescription(simple: boolean)`: Returns array of `{desc, ref, simple}`

### Eidolon
- **Purpose:** Represents a character's eidolon (constellation) with description and skill upgrades.
- **Key Properties:**
  - `id`, `rank`, `icon`, `name`, `description`, `skillsLevelUp`

### StatProperty, StatPropertyValue
- **StatProperty:** Static stat definition (type, name, icon, etc.)
- **StatPropertyValue:** Value for a stat (type, value, isPercent, statProperty)

### LightConeData, RelicData
- **LightConeData:** Static info for a light cone (id, name, rarity, skills, icon, etc.)
- **RelicData:** Static info for a relic (id, name, set, main stat, sub stats, icon, etc.)

### TextAssets, DynamicTextAssets, ImageAssets
- **TextAssets:** Localized text (with `.get(lang)` method)
- **DynamicTextAssets:** Localized text with parameters (for skill/traces descriptions)
- **ImageAssets:** Image URLs for icons, art, etc.

### StarRailUser
- **Purpose:** Represents a fetched user profile (UID, characters, stats, etc.)

---

## Example Data Types (from runCharacterExtraction)

### CharacterBaseStats
```ts
interface CharacterBaseStats {
    ascension: number;
    level: number;
    cost: { id: number; count: number }[];
    atkBase: number;
    hpBase: number;
    defBase: number;
    spdBase: number;
    critRate: number;
    critDmg: number;
    tauntBase: number;
}
```

### SkillData
```ts
interface SkillData {
    id: string;
    groupId: string;
    name: string;
    groupName: string;
    effectType: string;
    skillType: string;
    StanceDamageType: string;
    maxLevel: number;
    description: string;
    break: string;
    SPNeed: number;
    BPNeed: number;
    InitCoolDown: number;
    CoolDown: number;
    DelayRatio: number;
    SPMultipleRatio: number;
    ShowStanceList: number[];
    levels: SkillLevelData[];
    iconPath: string;
    treeNodeIconPath: string;
}

interface SkillLevelData {
    level: number;
    params: number[];
}
```

### TraceData
```ts
interface StatBonus {
    type: string;
    value: number;
    isPercent: boolean;
}

interface TraceData {
    id: number;
    name: string | null;
    description: string | null;
    iconPath: string | null;
    isUnlockedByDefault: boolean;
    ascensionRequirement: number | null;
    traceType: "Minor" | "Major";
    stats: StatBonus[];
    effectType?: string | null;
    paramList?: number[];
}
```

### EidolonExport
```ts
type EidolonExport = Omit<Eidolon, 'client' | '_data' | 'picture' | 'skillsLevelUp' | 'constructor'> & {
    paramList: number[];
    description: string | null;
    skillsLevelUp: {
        skillId: string | number;
        skillType: string;
        skillName: string;
        upgrade: number;
    }[];
};
```

---

## Usage Examples

### Fetching All Characters
```js
const client = new StarRail();
const characters = client.getAllCharacters();
console.log(characters.map(c => c.name.get('en')));
```

### Fetching Player Data
```js
const user = await client.fetchUser(800069903);
console.log(user);
```

### Building a Character Instance
```js
const characterData = client.getAllCharacters()[0];
const character = Character.builder()
  .character(characterData, true)
  .level(80)
  .ascension(6)
  .eidolons(0)
  .build(client);
console.log(character.stats);
```

### Extracting Skills, Traces, Eidolons
```js
const skills = extractCharacterSkills(characterInstance); // returns SkillData[]
const traces = extractTraces(characterInstance); // returns TraceData[]
const eidolons = extractEidolons(characterInstance); // returns EidolonExport[]
```

---

## Data Model Overview

- **CharacterData:** Static info (name, path, element, skills, traces, eidolons, icons)
- **Skill:** Skill definition (id, type, icon, maxLevel, etc.)
- **LeveledSkill:** Skill at a specific level (params, description)
- **SkillTreeNode:** Trace node (major/minor, stat bonus, unlock requirements)
- **Eidolon:** Eidolon/constellation (description, skill upgrades)
- **LightConeData:** Light cone static info
- **RelicData:** Relic static info
- **StatProperty/StatPropertyValue:** Stat definitions and values
- **TextAssets/DynamicTextAssets:** Localized text and descriptions
- **ImageAssets:** Icon and art URLs

----------------------------------------

TITLE: starrail - StarRail Client Initialization
DESCRIPTION: How to initialize the main StarRail client for accessing Honkai: Star Rail data, including cache configuration and language selection.
LANGUAGE: javascript
CODE:
```
const client = new StarRail({
  defaultLanguage: 'en',
  cacheDirectory: './.cache',
  showFetchCacheLog: true
});
```

----------------------------------------

TITLE: starrail - Fetching All Characters
DESCRIPTION: Retrieve all available character data using the StarRail client and print their names in English.
LANGUAGE: javascript
CODE:
```
const characters = client.getAllCharacters();
console.log(characters.map(c => c.name.get('en')));
```

----------------------------------------

TITLE: starrail - Building a Character Instance
DESCRIPTION: Use the builder pattern to create a Character instance at a specific level, ascension, and eidolon state.
LANGUAGE: javascript
CODE:
```
const character = Character.builder()
  .character(characterData, true)
  .level(80)
  .ascension(6)
  .eidolons(0)
  .build(client);
```

----------------------------------------

TITLE: starrail - Extracting Character Skills
DESCRIPTION: Example of extracting all skills for a character using the skill tree nodes and mapping them to a serializable structure.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
function extractCharacterSkills(characterInstance: Character): SkillData[] {
  const skillsOutput: SkillData[] = [];
  for (const treeNode of characterInstance.skillTreeNodes as LeveledSkillTreeNode[]) {
    if (!treeNode.levelUpSkills || treeNode.levelUpSkills.length === 0) continue;
    const baseSkillDefinition: Skill = treeNode.levelUpSkills[0];
    if (!baseSkillDefinition) continue;
    if (!skillTypeTranslation[baseSkillDefinition.skillType]) continue;
    const levelOneSkillLevelObj = new SkillLevel(1, 0);
    const firstLeveledSkill = baseSkillDefinition.getSkillByLevel(levelOneSkillLevelObj);
    let rawDesc = "";
    if (firstLeveledSkill) {
      rawDesc = safeGetText(firstLeveledSkill.description);
    }
    const leveledNode: LeveledSkillTreeNode = treeNode.getSkillTreeNodeByLevel(new SkillLevel(1, 0));
    const descArr = leveledNode.getFullDescription(false);
    //@ts-ignore
    const refData = descArr[0].ref._skillsData?.[0] || {};
    const skillData: SkillData = {
      id: String(baseSkillDefinition.id),
      groupId: String(treeNode.id),
      name: safeGetText(baseSkillDefinition.name),
      groupName: safeGetText(treeNode.name),
      effectType: effectTypeTranslation[baseSkillDefinition.effectType],
      skillType: skillTypeTranslation[baseSkillDefinition.skillType],
      StanceDamageType: elementTranslation[refData.StanceDamageType] ?? refData.StanceDamageType,
      maxLevel: baseSkillDefinition.maxLevel,
      description: rawDesc,
      break: String(refData.StanceDamageDisplay),
      SPNeed: refData.SPNeed?.Value ?? refData.SPNeed,
      BPNeed: refData.BPNeed?.Value ?? refData.BPNeed,
      InitCoolDown: refData.InitCoolDown,
      CoolDown: refData.CoolDown,
      DelayRatio: refData.DelayRatio?.Value ?? refData.DelayRatio,
      SPMultipleRatio: refData.SPMultipleRatio?.Value ?? refData.SPMultipleRatio,
      ShowStanceList: refData.ShowStanceList?.map(obj => obj.Value),
      levels: [],
      iconPath: baseSkillDefinition.skillIcon?.url,
      treeNodeIconPath: treeNode.icon?.url,
    };
    for (let currentLevelValue = 1; currentLevelValue <= baseSkillDefinition.maxLevel; currentLevelValue++) {
      const skillLevelObj: SkillLevel = new SkillLevel(currentLevelValue, 0);
      const leveledSkill: LeveledSkill = baseSkillDefinition.getSkillByLevel(skillLevelObj);
      if (!leveledSkill) {
        continue;
      }
      skillData.levels.push({
        level: leveledSkill.level.value,
        params: leveledSkill.paramList || []
      });
    }
    skillData.levels.sort((a, b) => a.level - b.level);
    skillsOutput.push(skillData);
  }
  return skillsOutput;
}
```

----------------------------------------

TITLE: starrail - SkillData Type Definition
DESCRIPTION: TypeScript interface for the structure of extracted skill data, including all relevant fields for serialization.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
interface SkillData {
  id: string;
  groupId: string;
  name: string;
  groupName: string;
  effectType: string;
  skillType: string;
  StanceDamageType: string;
  maxLevel: number;
  description: string;
  break: string;
  SPNeed: number;
  BPNeed: number;
  InitCoolDown: number;
  CoolDown: number;
  DelayRatio: number;
  SPMultipleRatio: number;
  ShowStanceList: number[];
  levels: SkillLevelData[];
  iconPath: string;
  treeNodeIconPath: string;
}
```

----------------------------------------

TITLE: starrail - TraceData Type Definition
DESCRIPTION: TypeScript interface for the structure of extracted trace (major/minor) data, including stat bonuses and requirements.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
interface TraceData {
  id: number;
  name: string | null;
  description: string | null;
  iconPath: string | null;
  isUnlockedByDefault: boolean;
  ascensionRequirement: number | null;
  traceType: "Minor" | "Major";
  stats: StatBonus[];
  effectType?: string | null;
  paramList?: number[];
}
```

----------------------------------------

TITLE: starrail - EidolonExport Type Definition
DESCRIPTION: TypeScript type for the structure of extracted eidolon data, omitting circular and non-serializable fields.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
type EidolonExport = Omit<Eidolon, 'client' | '_data' | 'picture' | 'skillsLevelUp' | 'constructor'> & {
  paramList: number[];
  description: string | null;
  skillsLevelUp: {
    skillId: string | number;
    skillType: string;
    skillName: string;
    upgrade: number;
  }[];
};
```

----------------------------------------

TITLE: starrail - Fetching Player Data
DESCRIPTION: Fetch a player's public Honkai: Star Rail profile by UID using the StarRail client.
LANGUAGE: javascript
CODE:
```
const user = await client.fetchUser(800069903);
console.log(user);
```

----------------------------------------

TITLE: starrail - Updating Star Rail Cache Data
DESCRIPTION: Update the local cache of Star Rail data to ensure the latest characters, light cones, and materials are available.
LANGUAGE: javascript
CODE:
```
const client = new StarRail({ showFetchCacheLog: true });
await client.cachedAssetsManager.fetchAllContents();
```

----------------------------------------

TITLE: starrail - CharacterBaseStats Type Definition
DESCRIPTION: TypeScript interface for the structure of extracted character base stats, including ascension, level, and stat values.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
interface CharacterBaseStats {
  ascension: number;
  level: number;
  cost: { id: number; count: number }[];
  atkBase: number;
  hpBase: number;
  defBase: number;
  spdBase: number;
  critRate: number;
  critDmg: number;
  tauntBase: number;
}
```

----------------------------------------

TITLE: starrail - Full extractCharacterSkills Function
DESCRIPTION: Complete implementation of the extractCharacterSkills function, which extracts all skills for a character and maps them to a serializable structure.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
function extractCharacterSkills(characterInstance: Character): SkillData[] {
    const skillsOutput: SkillData[] = [];
    for (const treeNode of characterInstance.skillTreeNodes as LeveledSkillTreeNode[]) {
        if (!treeNode.levelUpSkills || treeNode.levelUpSkills.length === 0) {
            continue;
        }
        const baseSkillDefinition: Skill = treeNode.levelUpSkills[0];
        if (!baseSkillDefinition) {
            continue;
        }
        if (!skillTypeTranslation[baseSkillDefinition.skillType]) {
            continue;
        }
        const levelOneSkillLevelObj = new SkillLevel(1, 0);
        const firstLeveledSkill = baseSkillDefinition.getSkillByLevel(levelOneSkillLevelObj);
        let rawDesc = "";
        if (firstLeveledSkill) {
            rawDesc = safeGetText(firstLeveledSkill.description);
        }
        const leveledNode: LeveledSkillTreeNode = treeNode.getSkillTreeNodeByLevel(new SkillLevel(1, 0));
        const descArr = leveledNode.getFullDescription(false);
        //@ts-ignore
        const refData = descArr[0].ref._skillsData?.[0] || {};
        const skillData: SkillData = {
            id: String(baseSkillDefinition.id),
            groupId: String(treeNode.id),
            name: safeGetText(baseSkillDefinition.name),
            groupName: safeGetText(treeNode.name),
            effectType: effectTypeTranslation[baseSkillDefinition.effectType],
            skillType: skillTypeTranslation[baseSkillDefinition.skillType],
            StanceDamageType: elementTranslation[refData.StanceDamageType] ?? refData.StanceDamageType,
            maxLevel: baseSkillDefinition.maxLevel,
            description: rawDesc,
            break: String(refData.StanceDamageDisplay),
            SPNeed: refData.SPNeed?.Value ?? refData.SPNeed,
            BPNeed: refData.BPNeed?.Value ?? refData.BPNeed,
            InitCoolDown: refData.InitCoolDown,
            CoolDown: refData.CoolDown,
            DelayRatio: refData.DelayRatio?.Value ?? refData.DelayRatio,
            SPMultipleRatio: refData.SPMultipleRatio?.Value ?? refData.SPMultipleRatio,
            ShowStanceList: refData.ShowStanceList?.map(obj => obj.Value),
            levels: [],
            iconPath: baseSkillDefinition.skillIcon?.url,
            treeNodeIconPath: treeNode.icon?.url,
        };
        for (let currentLevelValue = 1; currentLevelValue <= baseSkillDefinition.maxLevel; currentLevelValue++) {
            const skillLevelObj: SkillLevel = new SkillLevel(currentLevelValue, 0);
            const leveledSkill: LeveledSkill = baseSkillDefinition.getSkillByLevel(skillLevelObj);
            if (!leveledSkill) {
                continue;
            }
            skillData.levels.push({
                level: leveledSkill.level.value,
                params: leveledSkill.paramList || []
            });
        }
        skillData.levels.sort((a, b) => a.level - b.level);
        skillsOutput.push(skillData);
    }
    return skillsOutput;
}
```

----------------------------------------

TITLE: starrail - Full extractTraces Function
DESCRIPTION: Complete implementation of the extractTraces function, which extracts all trace nodes (major and minor) for a character and maps them to a serializable structure.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
function extractTraces(characterInstance: Character): TraceData[] {
    const tracesOutput: TraceData[] = [];
    for (const treeNode of characterInstance.skillTreeNodes as SkillTreeNode[]) {
        // Generalized: Skip Technique node by checking skillType
        if (
            treeNode.levelUpSkills &&
            treeNode.levelUpSkills.length > 0 &&
            treeNode.levelUpSkills[0].skillType === "Maze"
        ) continue;
        if (treeNode.maxLevel === 1) {
            const skillLevel = new SkillLevel(1, 0);
            const leveledNode = treeNode.getSkillTreeNodeByLevel(skillLevel);
            if (!leveledNode) {
                continue;
            }
            const descArr = leveledNode.getFullDescription(true);
            const detail = descArr[0]?.ref;
            const traceData: Partial<TraceData> = {
                id: treeNode.id,
                name: safeGetText(treeNode.name),
                iconPath: treeNode.icon?.url,
                isUnlockedByDefault: treeNode.isUnlockedByDefault,
                ascensionRequirement: safeNumber(leveledNode._data.AvatarPromotionLimit, null),
                stats: [],
            };
            if (detail instanceof StatPropertyValue) {
                traceData.traceType = "Minor";
                traceData.stats!.push({
                    type: String(detail.type),
                    value: detail.value,
                    isPercent: detail.isPercent
                });
                traceData.description = `${safeGetText(detail.statProperty?.name)} +${detail.isPercent ? `${detail.value * 100}%` : detail.value}`;
            } else if (detail instanceof LeveledSkill || !detail) {
                traceData.traceType = "Major";
                traceData.description = safeGetDynamicText(leveledNode.description, leveledNode.paramList) || safeGetText(leveledNode.description);
                traceData.paramList = leveledNode.paramList;
                if (detail instanceof LeveledSkill) {
                    traceData.effectType = effectTypeTranslation[detail.effectType];
                    traceData.description = safeGetDynamicText(detail.description, detail.paramList) || traceData.description;
                    if (!traceData.paramList || traceData.paramList.length === 0) {
                        traceData.paramList = detail.paramList;
                    }
                }
            } else {
                continue;
            }
            if (leveledNode.stats && leveledNode.stats.length > 0) {
                for (const statPropVal of leveledNode.stats) {
                    const alreadyAdded = traceData.stats!.some(s =>
                        s.type === String(statPropVal.type) && s.value === statPropVal.value
                    );
                    if (!alreadyAdded) {
                        traceData.stats!.push({
                            type: String(statPropVal.type),
                            value: statPropVal.value,
                            isPercent: statPropVal.isPercent
                        });
                    }
                }
            }
            tracesOutput.push(traceData as TraceData);
        }
    }
    return tracesOutput;
}
```

----------------------------------------

TITLE: starrail - Full extractEidolons Function
DESCRIPTION: Complete implementation of the extractEidolons function, which extracts all eidolons for a character and maps them to a serializable structure.
SOURCE: runCharacterExtraction.ts
LANGUAGE: typescript
CODE:
```
function extractEidolons(characterInstance: Character): EidolonExport[] {
    const eidolonsOutput: EidolonExport[] = [];
    if (!characterInstance.characterData || !characterInstance.characterData.eidolons) return eidolonsOutput;
    const skills = characterInstance.skills || [];
    const skillMap = new Map<string | number, Skill>();
    for (const skill of skills) {
        skillMap.set(skill.id, skill);
    }
    for (const eidolon of characterInstance.characterData.eidolons as Eidolon[]) {
        const paramList = (eidolon as any)._data?.Param?.map((p: any) => p.Value) || [];
        let description: string | null = null;
        if (eidolon.description instanceof DynamicTextAssets) {
            description = safeGetDynamicText(eidolon.description, paramList);
        } else {
            description = safeGetText(eidolon.description);
        }
        const skillsLevelUpArr: {
            skillId: string | number;
            skillType: string;
            skillName: string;
            upgrade: number;
        }[] = [];
        if (eidolon.skillsLevelUp && typeof eidolon.skillsLevelUp === 'object') {
            for (const key of Object.keys(eidolon.skillsLevelUp)) {
                const skillUp = eidolon.skillsLevelUp[key];
                const skill = skillUp.skill;
                skillsLevelUpArr.push({
                    skillId: skill.id,
                    skillType: skill && skill.skillType ? (skillTypeTranslation[skill.skillType] || skill.skillType) : '',
                    skillName: skill && skill.name ? safeGetText(skill.name) : '',
                    upgrade: skillUp.levelUp
                });
            }
        }
        eidolonsOutput.push({
            id: eidolon.id,
            rank: eidolon.rank,
            icon: eidolon.icon?.url,
            name: safeGetText(eidolon.name),
            description,
            paramList,
            skillsLevelUp: skillsLevelUpArr
        });
    }
    return eidolonsOutput.sort((a, b) => a.rank - b.rank);
}
```

----------------------------------------
