<template>
  <div class="editor">
    <h1>Editor</h1>
    <span>{{ user.displayName }}</span>
    <button @click="logout">Logout</button>
    <div class="editorWrapper">
      <div class="memoListWrapper">
        <div
          class="memoList"
          v-for="(memo, index) in memos"
          :key="index"
          @click="selectMemo(index)"
          :data-selected="index == selectedIndex"
        >
          <p class="memoTitle">{{displayTitle(memo.markdown)}}</p>
        </div>
        <button class="addMemoBtn" @click="addMemo">Add Memo</button>
        <button class="deleteMemoBtn" v-if="memos.length > 1" @click="deleteMemo">Delete</button>
        <button class="saveMemosBtn" @click="saveMemos">Save this memo</button>
      </div>
      <textarea class="markdown" v-model="memos[selectedIndex].markdown"></textarea>
      <div class="preview markdown-body" v-html="preview()"></div>
    </div>
  </div>
</template>

<script>
import marked from "marked";
export default {
  name: "editor",
  props: ["user"],
  // 初期データ
  data() {
    return {
      memos: [{ markdown: "" }],
      selectedIndex: 0
    };
  },
  // ページ読み込み時にデータの読み込み
  created: function() {
    firebase
      .database()
      .ref("memos/" + this.user.uid)
      // once: 1回だけの読み込み
      .once("value")
      .then(result => {
        // .val: 指定のパスのデータを取得
        if (result.val()) {
          this.memos = result.val();
        }
      });
  },
  methods: {
    logout: function() {
      firebase.auth().signOut();
    },
    addMemo: function() {
      this.memos.push({
        markdown: "No title"
      });
    },
    selectMemo: function(index) {
      this.selectedIndex = index;
    },
    preview: function() {
      return marked(this.memos[this.selectedIndex].markdown);
    },
    displayTitle: function(text) {
      //　split(/\n/): テキストを改行で分割
      return text.split(/\n/)[0];
    },
    deleteMemo: function() {
      this.memos.splice(this.selectedIndex, 1);
      if (this.selectedIndex > 0) {
        this.selectedIndex--;
      }
    },

    saveMemos: function() {
      firebase
        //　FB() => DBに接続し
        .database()
        //　refでデータを読み書きするパスを指定する。uidはFBでログインした際に発行されるID
        .ref("memos/" + this.user.uid)
        // setでデータを保存(update)
        .set(this.memos);
    }
  },
  mounted: function() {
    //ctrl S / command s => save
    document.onkeydown = e => {
      if (e.key == "s" && (e.metaKey || e.ctrlKey)) {
        this.saveMemos();
        return false;
      }
    };
  },
  // コンポネントがログアウトなどで削除されるされるタイミングで設定を消す
  beforeDestroy: function() {
    document.onkeydown = null;
  }
};
</script>
<style lang="scss" scoped>
.editorWrapper {
  display: flex;
}
.memoListWrapper {
  width: 20%;
  border-top: 1px solid #000;
}
.memoList {
  padding: 10px;
  box-sizing: border-box;
  text-align: left;
  border-bottom: 1px solid #000;
  &:nth-child(even) {
    background-color: #ccc;
  }
  &[data-selected="true"] {
    background-color: #ccf;
  }
}
.memoTitle {
  height: 1.5em;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
}
.addMemoBtn {
  margin-top: 20px;
}
.markdown {
  width: 40%;
  height: 500px;
}
.preview {
  width: 40%;
  text-align: left;
}
.deleteMemoBtn {
  margin: 10px;
}
</style>