<template>
  <div id="trade-view" style="height: 500px">
  </div>
</template>

<script>
import { widget as TvWidget } from '../../../static/tradeview/charting_library/charting_library.min.js'
import socket from './datafeeds/socket.js'
import datafeeds from './datafeeds/datafees.js'
import $ from 'jquery'
export default {
  data() {
    return {
      widget: null,
      socket: new socket(),
      datafeeds: new datafeeds(this),
      symbol: null,
      interval: null,
      cacheData: {},
      lastTime: null,
      getBarTimer: null,
      isLoading: true,
      buttons: [
        { title: "1分", resolution: "1", chartType: 1 },
        { title: "5分", resolution: "5", chartType: 1 },
        { title: "15分", resolution: "15", chartType: 1 },
        { title: "30分", resolution: "30", chartType: 1 },
        { title: "时", resolution: "60", chartType: 1 },
        { title: "日", resolution: "1D", chartType: 1 },
        { title: "周", resolution: "1W", chartType: 1 }
      ],
      studies: [],
      
    }
  },
  created() {
    this.socket.doOpen()
    this.socket.on('open', () => {
      // this.socket.send({ cmd: 'req', args: [`candle.M5.btcusdt}`, 1440, parseInt(Date.now() / 1000)] })
      this.socket.send({
      dataType: "731_KLINE_5M_XT_SXC",
      dataSize: 500,
      action: "ADD"
    })
    })
    this.socket.on('message', this.onMessage)
  },
  methods: {
    init(symbol = 'BTCUSDT', interval = 5) {
      if (!this.widget) {
        this.widget = new TvWidget({
          symbol: symbol,
          interval: interval,
          // fullscreen: true,
          container_id: 'trade-view',
          datafeed: this.datafeeds,
          library_path: '/static/tradeview/charting_library/',
          enabled_features: [],
          timezone: 'Asia/Shanghai',
          locale: 'zh',
          debug: false,
          autosize: true,
          toolbar_bg: "#20334d",
           overrides: {
            "paneProperties.background": "#24334b",
            "paneProperties.vertGridProperties.color": "#454545",
            "paneProperties.horzGridProperties.color": "#454545",
            "symbolWatermarkProperties.transparency": 90,
            "scalesProperties.textColor": "#AAA"
          },
          disabled_features: [
            "use_localstorage_for_settings",
            "show_interval_dialog_on_key_press",
            "symbol_search_hot_key",
            "study_dialog_search_control",
            "edit_this.buttons_in_legend",
            "context_menus",
            "volume_force_overlay",
            "header_interval_dialog_button",
            "header_symbol_search",
            "header_saveload",
            "header_screenshot",
            "header_chart_type",
            "header_compare",
            "header_undo_redo",
            "header_resolutions",
          ],
          studies_overrides: {
            "volume.volume.color.0": "#fe4761",
            "volume.volume.color.1": "#3fcfb4",
            "volume.volume.transparency": 75
          },
          custom_css_url: "chart.css"
        })
        this.symbol = symbol
        this.interval = interval

        this.interval = interval;
        var thats = this.widget;
        var _this = this;
        var chartType = 1;
        thats.onChartReady(function() {
          //   //设置均线种类 均线样式
          // _this.createStudy();
          //   //生成时间按钮
          
          _this.createButton(_this.buttons);
            thats.chart().setChartType(chartType);
            _this.toggleStudy(chartType);
        });
      }
    },
    createStudy() {
      var thats = this.widget;
      var id = thats
        .chart()
        .createStudy("Moving Average", true, true, [5], null, {
          "Plot.color": "rgb(150, 95, 196)"
        });
      this.studies.push(id);
      id = thats.chart().createStudy("Moving Average", true, true, [10], null, {
        "Plot.color": "rgb(116,149,187)"
      });
      this.studies.push(id);
      id = thats.chart().createStudy("Moving Average", true, true, [20], null, {
        "plot.color": "rgb(58,113,74)"
      });
      this.studies.push(id);
      id = thats.chart().createStudy("Moving Average", true, true, [30], null, {
        "plot.color": "rgb(118,32,99)"
      });
      this.studies.push(id);
    },
    toggleStudy(chartType) {
      var thats = this.widget;
      var state = chartType == 3 ? 0 : 1;
      for (var i = 0; i < this.studies.length; i++) {
        thats
          .chart()
          .getStudyById(this.studies[i])
          .setVisible(state);
      }
    },
    createButton() {
      var thats = this.widget;
      var _this = this
      for (var i = 0; i < this.buttons.length; i++) {
        (button => {
          thats
            .createButton()
            .attr("title", button.title)
            .addClass(i == 1 ? `mydate active`: `mydate`)
            .text(button.title)
            .on("click", (e) => {     
              let sb = e.toElement.parentNode.parentNode.children;
              for (let i = 0; i < sb.length; i++) {
                if(sb[i].children[0].className.indexOf('mydate') != -1) {
                  sb[i].children[0].className = "button mydate"
                }
              }
              e.toElement.className = e.toElement.className + ' active';          
            thats.chart().setResolution(button.resolution);
            });
        })(this.buttons[i]);
      }
    },
    sendMessage(data) {
      if (this.socket.checkOpen()) {
        this.socket.send(data)
      } else {
        this.socket.on('open', () => {
          this.socket.send(data)
        })
      }
    },
    unSubscribe(interval) {
       console.log('之前',this.interval);
      switch (interval) {
          case '1D':
          this.sendMessage({
              dataType: `731_KLINE_${this.interval}_XT_SXC`,
              dataSize: 500,
              action: "DEL"
          })
          break;
          case '1W':
            this.sendMessage({
              dataType: `731_KLINE_${this.interval}_XT_SXC`,
              dataSize: 500,
              action: "DEL"
          })
          break;
          case '60':
            this.sendMessage({
              dataType: `731_KLINE_1H_XT_SXC`,
              dataSize: 500,
              action: "DEL"
          })
          break;
           case 'D':
            this.sendMessage({
              dataType: `731_KLINE_1W_XT_SXC`,
              dataSize: 500,
              action: "DEL"
          })
          break;
          default:
          this.sendMessage({
              dataType: `731_KLINE_${this.interval}M_XT_SXC`,
              dataSize: 500,
              action: "DEL"
          })
          break;
      }
      
    
      // if (interval < 60) {
      //   this.sendMessage({ cmd: 'unsub', args: [`candle.M${interval}.${this.symbol.toLowerCase()}`, 1440, parseInt(Date.now() / 1000)] })
      // } else if (interval >= 60) {
      //   this.sendMessage({ cmd: 'unsub', args: [`candle.H${interval / 60}.${this.symbol.toLowerCase()}`, 1440, parseInt(Date.now() / 1000)] })
      // } else {
      //   this.sendMessage({ cmd: 'unsub', args: [`candle.D1.${this.symbol.toLowerCase()}`, 207, parseInt(Date.now() / 1000)] })
      // }
    },
    subscribe() {
      console.log('之后',this.interval);
      
      switch (this.interval) {
        case '1D':
          this.sendMessage({
              dataType: `731_KLINE_${this.interval}_XT_SXC`,
              dataSize: 500,
              action: "ADD"
          })
          break;
          case '1W':
            this.sendMessage({
              dataType: `731_KLINE_${this.interval}_XT_SXC`,
              dataSize: 500,
              action: "ADD"
          })
          break;
          case '60':
            this.sendMessage({
              dataType: `731_KLINE_1H_XT_SXC`,
              dataSize: 500,
              action: "ADD"
          })
          break;
          case 'D':
            this.sendMessage({
              dataType: `731_KLINE_1W_XT_SXC`,
              dataSize: 500,
              action: "ADD"
          })
          break;
        default:
          this.sendMessage({
              dataType: `731_KLINE_${this.interval}M_XT_SXC`,
              dataSize: 500,
              action: "ADD"

          })
          break;
      }
      
      // if (this.interval < 60) {
      //   this.sendMessage({ cmd: 'sub', args: [`candle.M${this.interval}.${this.symbol.toLowerCase()}`] })
      // } else if (this.interval >= 60) {
      //   this.sendMessage({ cmd: 'sub', args: [`candle.H${this.interval / 60}.${this.symbol.toLowerCase()}`] })
      // } else {
      //   this.sendMessage({ cmd: 'sub', args: [`candle.D1.${this.symbol.toLowerCase()}`] })
      // }
    },
    onMessage(data) {
      if (data[0] instanceof Array) {
        const list = []
        const ticker = `${this.symbol}-${this.interval}`
        data.forEach(function (v) {
          list.unshift({
            // time: this.interval !== 'D' || this.interval !== '1D' ? element.id * 1000 : element.id,
            time: Number(v[3] * 1000),
            close: Number(v[7]),
            open: Number(v[4]),
            high: Number(v[5]),
            low: Number(v[6]),
            volume: Number(v[13])
          })
        }, this)
        this.cacheData[ticker] = list
        this.lastTime = list[list.length - 1].time
        // this.subscribe()
      }else  {
        // console.log(' >> sub:', data.type)
        this.datafeeds.barsUpdater.updateData()
        const ticker = `${this.symbol}-${this.interval}`
        const barsData = {
          // time: data.id * 1000,
             time: data[3] * 1000,
            close: data[7],
            open: data[4],
            high: data[5],
            low: data[6],
            volume: data[13]
        }
        if (barsData.time >= this.lastTime && this.cacheData[ticker] && this.cacheData[ticker].length) {
          this.cacheData[ticker][this.cacheData[ticker].length - 1] = barsData
        }
      }
    },
    getBars(symbolInfo, resolution, rangeStartDate, rangeEndDate, onLoadedCallback) {
      // console.log(' >> :', rangeStartDate, rangeEndDate)
      
      if (this.interval != resolution) {
        this.unSubscribe(this.interval)
        this.interval = resolution
        this.subscribe();
        console.log('bars',this.interval ,resolution);
      }
      const ticker = `${this.symbol}-${this.interval}`
      if (this.cacheData[ticker] && this.cacheData[ticker].length) {
        this.isLoading = false
        const newBars = []
        this.cacheData[ticker].forEach(item => {
          if (item.time >= rangeStartDate * 1000 && item.time <= rangeEndDate * 1000) {
            newBars.push(item)
          }
        })
        onLoadedCallback(newBars)
      } else {
        const self = this
        this.getBarTimer = setTimeout(function () {
          self.getBars(symbolInfo, resolution, rangeStartDate, rangeEndDate, onLoadedCallback)
        }, 10)
      }
    }
  }
}
</script>

