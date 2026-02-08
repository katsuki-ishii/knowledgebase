---
base: "[[Vue.js.base]]"
作成者: Katsubo Katsubo
カテゴリー:
  - Vue.js
作成日時: 2025-11-23T14:25:00
aliases: [Vueコード規約, Vue規約, コード規約, composables, Pinia]
---
## 1. 基本原則

- ロジックはすべて [[基礎 Part 16 Vueコード規約|composables]] に集約する。
- [[基礎 Part 30 コンポーネント|コンポーネント]]は UI と[[基礎 Part 15 イベントオブジェクト|イベント]]配線に特化させる。
- [[基礎 Part 10 setup構文|setup]] は長くなりすぎないようにし、肥大化した部分は [[基礎 Part 16 Vueコード規約|composables]] へ移動する。
- Options API の構造（data、methods など）は残さない。
- 関数・変数は用途が一読で分かる命名にする。

## 2. [[基礎 Part 1 ディレクトリ構造|ディレクトリ構造]]の原則

NeXt の開発方針と、実際の画面改修での扱いやすさを踏まえ、**ページ単位でビュー配下に UI を集約する構造**を推奨する。

### 推奨構造

```javascript
src/
  assets/
  components/
    common/        # 複数画面で再利用される UI（ボタン、モーダル等）
    layout/        # ナビバー、サイドバーなどのレイアウト系

  composables/     # ロジック（状態管理・副作用・計算）を集約
    user/
    chat/
    gpt/
    shared/

  services/        # API 呼び出し専用レイヤー
    http/
    userService.js
    chatService.js
    gptService.js

  stores/          # 画面横断で共有する state（必要に応じて同一画面内でも使用可）
    userStore.js
    viewStore.js
    chatStore.js

  router/
    index.js

  views/           # 画面単位のコンテナ。UI と composables 呼び出しを担当
    featureA/
      FeatureA.vue          # ページ本体（メインビュー）
      components/           # featureA 専用の UI（ページ固有の Atom/Molecule）
        ComponentA.vue
        ComponentB.vue
        complex/
          SubCompA.vue
          SubCompB.vue
    featureB/
      FeatureB.vue
      components/
        ...

  App.vue
  main.js

```

### この構造を推奨する理由

- **画面改修時に必要なファイルがすべて views 配下にまとまっているため探しやすい。**
- ページ専用 UI を components/feature/ に置くより、**編集対象が空間的に近いため生産性が高い**。
- ページ固有[[基礎 Part 30 コンポーネント|コンポーネント]]が拡張されても、views 内で階層化できるので **スケールしやすい**。
- 再利用可能 UI は components/common に昇格させることで役割が明確になる。


### [[基礎 Part 5 reactive関数|reactive]] を使うケース

- フォーム入力のように複数の値がセットで扱われる場合
- “オブジェクトとしてのまとまり”が強い状態

```javascript
const form = reactive({
  name: '',
  age: null,
  email: ''
})

```

### チーム向け追加ガイドライン

- 深いネストが必要な場合は [[基礎 Part 5 reactive関数|reactive]] を避ける（追跡が難しくなるため）。
- [[基礎 Part 5 reactive関数|reactive]] を返す場合は構造が固定されていることが前提となる。
- [[基礎 Part 16 Vueコード規約|composables]] の返り値は [[基礎 Part 4 ref関数|ref]] を基本とし、必要最小限だけ [[基礎 Part 5 reactive関数|reactive]] を返す。
- API の生データは [[基礎 Part 4 ref関数|ref]] に入れて、加工は [[基礎 Part 19 computed|computed]] で行う。

---

## 3. [[基礎 Part 16 Vueコード規約|composables]] の設計方針

- [[基礎 Part 16 Vueコード規約|composables]] は必ず関数として定義し、必要な値・関数のみ返す。
- UI や DOM に依存するコードは含めない。
- サービス（API 呼び出し）を内部で使用してもよいが、state 管理は composable 側で行う。
- 必要であれば機能別フォルダで整理する。

### composable の例

```javascript
// src/composables/user/useUser.js
import { ref, computed } from 'vue'
import { getUser, updateUser as updateUserApi } from '@/services/userService'

export function useUser() {
  const user = ref(null)
  const isLoading = ref(false)

  async function loadUser() {
    isLoading.value = true
    try {
      user.value = await getUser()
    } finally {
      isLoading.value = false
    }
  }

  async function updateUser(payload) {
    await updateUserApi(payload)
    user.value = payload
  }

  const fullName = computed(() => {
    if (!user.value) return ''
    return `${user.value.first} ${user.value.last}`
  })

  return {
    user,
    isLoading,
    fullName,
    loadUser,
    updateUser
  }
}

```

## 4. services の設計方針

- services は API 呼び出し専用にする。
- axios などの HTTP クライアント設定は src/services/http/ に集約する。
- 返り値の整形以上のロジックは [[基礎 Part 16 Vueコード規約|composables]] へ寄せる。

### service の例

```javascript
// src/services/userService.js
import http from './http/axios'

export function getUser() {
  return http.get('/user')
}

export function updateUser(payload) {
  return http.post('/user/update', payload)
}

```

## 5. [[基礎 Part 16 Vueコード規約|Pinia]] ストアの方針

- 画面を跨いで必要な state を保持することを基本とする。
- ただし **同一画面内の複数[[基礎 Part 30 コンポーネント|コンポーネント]]間で状態共有が必要で、[[基礎 Part 36 defineProps()|props]] 受け渡しが複雑になる場合は [[基礎 Part 16 Vueコード規約|Pinia]] を使用してよい**。
- [[基礎 Part 16 Vueコード規約|composables]] はローカルな機能単位の状態保持に使う。
- store へ入れる state は「共有したい理由」が明確であることを条件とする。

### [[基礎 Part 16 Vueコード規約|Pinia]] を使うべきケース

- 画面を跨いで使うユーザー情報や設定値。
- 同一画面内で複数[[基礎 Part 30 コンポーネント|コンポーネント]]間にまたがる状態を共有する必要があり、[[基礎 Part 36 defineProps()|props]] バケツリレーが煩雑になる場合。
- 複雑な UI 状態（タブ、フィルター、選択状態など）を複数[[基礎 Part 30 コンポーネント|コンポーネント]]で扱う場合。

### [[基礎 Part 16 Vueコード規約|Pinia]] を使わない方がよいケース

- 単一[[基礎 Part 30 コンポーネント|コンポーネント]]内で完結する状態。
- 一時的な UI 状態（モーダル開閉、ローカルな検索キーワードなど）。
- [[基礎 Part 16 Vueコード規約|composables]] で簡単に共有できる程度の軽い状態。

### ストアの例

```javascript
// src/stores/userStore.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    isFetching: false
  }),
  actions: {
    setUser(payload) {
      this.user = payload
    },
    setFetching(flag) {
      this.isFetching = flag
    }
  }
})

```

## 6. [[基礎 Part 30 コンポーネント|コンポーネント]]の設計. [[基礎 Part 30 コンポーネント|コンポーネント]]の設計

- 再利用可能な UI 部品は components/common に配置する。
- layout 系（ヘッダー、サイドバー）は components/layout に分離する。
- 特定ページに属する UI 部品は components/feature 下に置く。
- [[基礎 Part 30 コンポーネント|コンポーネント]]にはビジネスロジックを入れず [[基礎 Part 16 Vueコード規約|composables]] から取得する。

### [[基礎 Part 30 コンポーネント|コンポーネント]]例

```javascript
<script setup>
import { ref, onMounted } from 'vue'
import { useUser } from '@/composables/user/useUser'

const { user, loadUser, updateUser } = useUser()
const isModalOpen = ref(false)

function openModal() {
  isModalOpen.value = true
}

function save() {
  updateUser(user.value)
  isModalOpen.value = false
}

onMounted(loadUser)
</script>

<template>
  <div>
    <p>{{ user?.name }}</p>
    <button @click="openModal">編集</button>
    <EditModal v-if="isModalOpen" @save="save" @close="isModalOpen = false" />
  </div>
</template>

```

## 7. [[基礎 Part 10 setup構文|setup]] の構造

以下の順番を統一的に採用する。

1. [[基礎 Part 11 ESM　Import＆Export|import]]
2. state（[[基礎 Part 4 ref関数|ref]], [[基礎 Part 5 reactive関数|reactive]]）
3. [[基礎 Part 19 computed|computed]]
4. methods（関数）
5. [[基礎 Part 23 watch|watch]]
6. lifecycle hooks
7. return

## 8. [[基礎 Part 23 watch|watch]] と [[基礎 Part 19 computed|computed]] の方針

- [[基礎 Part 19 computed|computed]] は値の導出専用とし、[[基礎 Part 21 副作用と純粋関数|副作用]]を入れない。
- [[基礎 Part 23 watch|watch]] は[[基礎 Part 21 副作用と純粋関数|副作用]]が必要な場合にのみ使う。
- 可能であれば[[基礎 Part 15 イベントオブジェクト|イベント]]伝播や明示的な関数呼び出しで代替する。

## 9. [[基礎 Part 36 defineProps()|props]] / emits

- [[基礎 Part 36 defineProps()|defineProps]] / [[基礎 Part 35 コンパイル時マクロ|defineEmits]] を用いる。
- [[基礎 Part 36 defineProps()|props]] のデフォルト値は必要に応じて親で明示する。
- emits は UI の動作を表す名前にする（save, close など）。

## 10. views の方針

- ページの読み込みや画面遷移に関する最小限の処理だけを書く。
- ロジックは [[基礎 Part 16 Vueコード規約|composables]] へ逃がす。
- 不要な再レンダリングを避けるため state をストアに過剰に置かない。

## 11. [[基礎 Part 4 ref関数|ref]] と [[基礎 Part 5 reactive関数|reactive]] の使い分け指針


Vue の思想である「明確で予測可能なリアクティビティ」を保つため、[[基礎 Part 4 ref関数|ref]] と [[基礎 Part 5 reactive関数|reactive]] の使い分けはチームで統一する。

基本原則


- 単一値（文字列、数値、真偽値など）は [[基礎 Part 4 ref関数|ref]] を使う。
- 複数プロパティで1つの意味を成す“まとまり”は [[基礎 Part 5 reactive関数|reactive]] を使う。
- 返り値として UI から利用されることを考えると [[基礎 Part 4 ref関数|ref]] の方が扱いやすい。
- [[基礎 Part 5 reactive関数|reactive]] は便利だが、状態追跡が複雑になるため必要なときだけ使う。

[[基礎 Part 4 ref関数|ref]] を使うケース
- フラグ（モーダル開閉など）
- 数値・文字列など単純な状態
- API レスポンスの保持（構造が安定しない場合）

## 12. レビュー観点

- [[基礎 Part 10 setup構文|setup]] の構成順序は守られているか。
- ロジックが [[基礎 Part 16 Vueコード規約|composables]] へ適切に分離されているか。
- 命名が用途を的確に表しているか。
- services が API 呼び出し専用として保たれているか。
- views や components が肥大化していないか。
- stores に余計なロジックが入っていないか。

## 13. 肥大化判断基準（[[基礎 Part 30 コンポーネント|コンポーネント]]・[[基礎 Part 16 Vueコード規約|composables]]・stores）

### [[基礎 Part 30 コンポーネント|コンポーネント]]（views / components）

- [[基礎 Part 10 setup構文|setup]] が 150 行を超えた場合は[[基礎 Part 13 分割代入|分割]]を検討する。
- 関数が 10 個を超える場合は責務過多と判断する。
- template が 200 行を超える場合は部分[[基礎 Part 30 コンポーネント|コンポーネント]]化を検討する。
- [[基礎 Part 36 defineProps()|props]] が 8 個以上の場合は[[基礎 Part 30 コンポーネント|コンポーネント]][[基礎 Part 13 分割代入|分割]]を検討する。

### [[基礎 Part 16 Vueコード規約|composables]]

- ファイルが 200 行を超える場合はロジック[[基礎 Part 13 分割代入|分割]]を検討する。
- return する変数や関数が 11 個以上になった場合は責務を[[基礎 Part 13 分割代入|分割]]する。
- [[基礎 Part 23 watch|watch]] が 3 つ以上ある場合はロジックが集中しすぎている可能性がある。

### stores（[[基礎 Part 16 Vueコード規約|Pinia]]）

- state が 10 項目以上ある場合は store の責務過多と判断する。
- actions が 10 個以上ある場合はロジックを [[基礎 Part 16 Vueコード規約|composables]] に移すことを検討する。
- 単一画面でしか使わない state は [[基礎 Part 16 Vueコード規約|composables]] または [[基礎 Part 30 コンポーネント|component]] 側に移動する。

これらの基準をもとに、[[基礎 Part 30 コンポーネント|コンポーネント]]や [[基礎 Part 16 Vueコード規約|composables]]、store の責務が適切に分担されているかを確認することを推奨する。
