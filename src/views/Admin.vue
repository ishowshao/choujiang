<template>
  <div>
    <h1>ADMIN</h1>
    <div style="display: flex">
      <div style="width: 40vw">
        <div>设置</div>
        <el-form size="small" label-width="150px">
          <el-form-item label="本轮抽取">
            <el-input-number v-model="count"></el-input-number>&nbsp;位&nbsp;
            <el-button @click="save">保存</el-button>
            <el-button @click="init">开始新一轮</el-button>
          </el-form-item>
          <el-form-item label="控制抽奖">
            <el-button @click="start">开始</el-button>
            <el-button @click="stop">停</el-button>
          </el-form-item>
          <el-form-item label="人员名单">
            <el-input v-model="names" type="textarea"></el-input>
          </el-form-item>
          <el-form-item label="允许重复中奖">
            <el-switch v-model="repeat"></el-switch>
          </el-form-item>
          <el-form-item label="初始文案">
            <el-input v-model="initText"></el-input>
          </el-form-item>
          <el-form-item label="初始文案字体大小">
            <el-input-number v-model="initTextFontSize"></el-input-number>
          </el-form-item>
          <el-form-item label="页面背景图">
            <el-input v-model="backgroundImage"></el-input>
          </el-form-item>
          <el-form-item label="抽奖按钮文本1">
            <el-input v-model="buttonText"></el-input>
          </el-form-item>
          <el-form-item label="抽奖按钮文本2">
            <el-input v-model="buttonText2"></el-input>
          </el-form-item>
          <el-form-item label="抽奖按钮图片">
            <el-input v-model="buttonImage" placeholder="请填写图片URL"></el-input>
          </el-form-item>
          <el-form-item label="抽奖按钮字体大小">
            <el-input-number v-model="buttonFontSize"></el-input-number>
          </el-form-item>
          <el-form-item label="抽奖按钮字体颜色">
            <el-color-picker v-model="buttonColor"></el-color-picker>
          </el-form-item>
          <el-form-item label="抽奖按钮宽度">
            <el-input-number v-model="buttonWidth"></el-input-number>
          </el-form-item>
          <el-form-item label="抽奖按钮高度">
            <el-input-number v-model="buttonHeight"></el-input-number>
          </el-form-item>
          <el-form-item label="抽奖按钮偏移X">
            <el-input-number v-model="buttonTranslateX"></el-input-number>
          </el-form-item>
          <el-form-item label="抽奖按钮偏移Y">
            <el-input-number v-model="buttonTranslateY"></el-input-number>
          </el-form-item>
          <el-form-item label="中奖人员字体大小">
            <el-input-number v-model="winnerFontSize"></el-input-number>
          </el-form-item>
          <el-form-item label="中奖人员字体颜色">
            <el-color-picker v-model="winnerColor"></el-color-picker>
          </el-form-item>
          <el-form-item label="中奖人员偏移X">
            <el-input-number v-model="winnerTranslateX"></el-input-number>
          </el-form-item>
          <el-form-item label="中奖人员偏移Y">
            <el-input-number v-model="winnerTranslateY"></el-input-number>
          </el-form-item>
          <el-form-item label="">
            <el-button @click="save">保存</el-button>
          </el-form-item>
        </el-form>
      </div>
      <div style="flex: 1; padding-left: 20px;">
        <div>中奖信息</div>
          <el-table :data="selection" style="width: 100%">
            <el-table-column type="index" width="50"></el-table-column>
            <el-table-column prop="time" label="时间" width="180">
              <template slot-scope="scope">{{new Date(scope.row.time).toLocaleString()}}</template>
            </el-table-column>
            <el-table-column prop="initText" label="文案" width="180"></el-table-column>
            <el-table-column prop="select" label="中奖人">
              <template slot-scope="scope">{{scope.row.select.join(',')}}</template>
            </el-table-column>
            <el-table-column prop="valid" label="状态" width="100">
              <template slot-scope="scope">
                <span>{{scope.row.valid ? '有效' : '已作废'}}</span>
              </template>
            </el-table-column>
            <el-table-column label="操作" width="100">
              <template slot-scope="scope">
                <el-button @click="invalid(scope.row)" type="text" size="small">作废</el-button>
              </template>
            </el-table-column>
          </el-table>
      </div>
    </div>

  </div>
</template>
<script>
export default {
  data() {
    return {
      count: 0,
      names: '',
      repeat: false,
      initText: '',
      initTextFontSize: 100,
      backgroundImage: '',
      buttonText: '开始',
      buttonText2: '结束',
      buttonImage: '',
      buttonFontSize: 16,
      buttonColor: '#000000',
      buttonWidth: 80,
      buttonHeight: 60,
      buttonTranslateX: 0,
      buttonTranslateY: 0,
      winnerFontSize: 40,
      winnerColor: '#000000',
      winnerTranslateX: 0,
      winnerTranslateY: 0,
      action: null,
      selection: [],
    };
  },
  methods: {
    save() {
      const date = new Date();
      const config = JSON.parse(JSON.stringify(this.$data));
      config.timestamp = date.toLocaleString();
      delete config.selection;
      localStorage.setItem('config', JSON.stringify(config));
    },
    init() {
      this.action = 'init';
      this.save();
      this.action = null;
    },
    start() {
      this.action = 'start';
      this.save();
      this.action = null;
    },
    stop() {
      this.action = 'stop';
      this.save();
      this.action = null;
    },
    invalid(row) {
      row.valid = false;
      localStorage.setItem('selection', JSON.stringify(this.selection));
    }
  },
  mounted() {
    const config = JSON.parse(localStorage.getItem('config') || '{}');
    Object.assign(this.$data, config);
  },
  created() {
    let selection = localStorage.getItem('selection');
    if (!selection) {
      localStorage.setItem('selection', JSON.stringify([]));
    }
    this.selection = JSON.parse(localStorage.getItem('selection'));
    window.addEventListener('storage', () => {
      console.log('receive message');
      this.selection = JSON.parse(localStorage.getItem('selection'));
    });
  }
};
</script>
