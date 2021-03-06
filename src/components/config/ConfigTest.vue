<template>
  <span>
    <el-popover v-model="testPopoverVisible" placement="bottom">
      <span slot="reference">
        <el-button v-if="!testRunLoading" type="primary" plain size="medium">
          Test
          <i v-if="!testPopoverVisible" class="el-icon-arrow-down el-icon-right" />
          <i v-if="testPopoverVisible" class="el-icon-arrow-up el-icon-right" />
        </el-button>
        <el-button v-else type="primary" plain disabled size="medium">Testing...</el-button>
      </span>
      <span class="m-e-sm">Test over previous</span>
      <ElastalertTimePicker
        :unit="Object.keys(testTime)[0]"
        :amount="Object.values(testTime)[0]"
        @input="updateTestTime" />
      <el-button class="m-w-med" type="primary" plain @click="runTest">Run</el-button>
    </el-popover>

    <ExpandableAlert
      v-if="testRunError"
      :contents="testRunError"
      title="Test run error"
      type="error"
      class="m-n-med"
    />

    <el-alert
      v-if="testRunResult &&
        testRunResult.writeback &&
      testRunResult.writeback.elastalert_status"
      :closable="false"
      type="success"
      show-icon
      title="Test run finished"
      class="m-n-med">
      <div>
        This rule would result in
        {{ testRunResult.writeback.elastalert_status.matches || 0 }}
        alert triggers over the last
        <ElastalertTimeView :time="testTime" />
      </div>
    </el-alert>

    <el-alert
      v-if="testRunLoading && messages.length"
      :closable="false"
      title="Test running"
      class="el-alert-loading m-n-med"
      type="info">
      <el-container>
        <div>
          <i class="el-icon-loading" />
        </div>
        <div style="flex: 1;">
          {{ messages.slice(-1)[0].replace('INFO:elastalert:', '') }}
        </div>
        <div>
          <el-button type="info" plain @click="cancelTestRun">Cancel</el-button>
        </div>
      </el-container>
    </el-alert>
  </span>
</template>

<script>
import moment from 'moment-timezone';
import { logger } from '@/lib/logger.js';

export default {
  props: ['valid'],

  data() {
    return {
      testTime: { hours: 1 },
      testPopoverVisible: false,
      messages: [],
      debugMessages: [],
      testRunResult: '',
      testRunError: '',
      testRunLoading: false
    };
  },

  destroyed() {
    this.$disconnect();
  },

  mounted() {
    this.$options.sockets.onmessage = ev => {
      let payload = JSON.parse(ev.data);

      if (payload.event === 'progress') {
        if (payload.data.startsWith('INFO:elastalert:')) {
          this.messages.push(payload.data);
        } else {
          this.debugMessages.push(payload.data);
          this.testRunError = this.debugMessages.join('\n');
        }
      } else if (payload.event === 'result') {
        try {
          this.testRunResult = JSON.parse(payload.data);
        } catch (error) {
          logger().error({ error: payload.data });
          this.testRunError = payload.data;
        } finally {
          this.testRunLoading = false;
          this.$disconnect();
        }
      } else if (payload.event === 'done') {
        this.testRunLoading = false;
        this.$disconnect();
      }
    };
  },

  methods: {
    updateTestTime(value) {
      this.testTime = value;
      this.testRunResult = '';
    },

    cancelTestRun() {
      this.$disconnect();
    },

    runTest() {
      this.testPopoverVisible = false;
      this.$emit('validate');

      this.$nextTick(() => {
        if (!this.valid) return;

        this.$connect();

        let rule = this.$store.getters['config/yaml'](true);

        this.testRunLoading = true;
        this.testRunResult = '';
        this.debugMessages = [];
        this.testRunError = '';
        this.messages = [];
        this.messages.push('Starting test run...');

        let start = moment().subtract(Object.values(this.testTime)[0], Object.keys(this.testTime)[0]).toISOString();

        let options = {
          testType: 'all',
          start,
          end: 'NOW',
          alert: false,
          format: 'json',
          maxResults: 1
        };

        this.$socket.onopen = () => {
          this.$socket.sendObj({ rule, options });
        };

        this.$socket.onclose = () => {
          this.testRunLoading = false;
        };
      });
    }
  }
};
</script>
