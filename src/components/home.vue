<template src="./home.html"></template>

<script>
  'use strict';

  export default {
    data() {
      return {
        stocksToSearch: 'FB,MSFT,r,t,f,e,w,q,',
        stocksData: [],
        selectedCounter: 0,
        error: [],
        displayError: false,
      }
    },
    mounted() {
      this.drawGraph();
    },
    methods: {

      // Send get request to yahoo and update stocks table
      getStocksData() {
        this.$http.get(`http://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20yahoo.finance.quotes%20where%20symbol%20IN%20("${this.stocksToSearch}")&format=json&env=store%3A%2F%2Fdatatables.org%2Falltableswithkeys`).then(response => {
          if (response.body.query.count === 1) response.body.query.results.quote = [response.body.query.results.quote];

          // translate JSON to smaller relevant objects
          response.body.query.results.quote.forEach(stock => {

            // check if recieved data for current stock
            if (!stock.Currency) this.error.push(`Could not find stock "${stock.symbol}"`);

            // check if current stock doesn't exist already in stocksData
            else {
              let isNewStock =
                this.stocksData.every(stockData => {
                  return stock.symbol.toUpperCase() !== stockData.symbol.toUpperCase()
                });
  
              if (isNewStock) {
                let {symbol, Name, PreviousClose, YearLow, YearHigh, ChangeFromYearLow, ChangeFromYearHigh} = stock;
                symbol = symbol.toUpperCase();
                let ChangeFromYearLowPercent = ChangeFromYearLow * 100 / YearLow;
                let ChangeFromYearHighPercent = ChangeFromYearHigh * 100 / YearHigh;
                this.stocksData.push({ symbol, Name, PreviousClose, YearLow, YearHigh, ChangeFromYearLow, ChangeFromYearHigh, ChangeFromYearLowPercent, ChangeFromYearHighPercent, selected: false });
              }
            }
          });

          // notify user error when searched for wrong stock
          if (this.error.length) {
            console.log('Error! ', this.error);
            this.error = [];
          }
        }, response => {
          console.log('There was an error: ', response);
        });
        this.stocksToSearch = '';
      },

      // When click on a table row
      toggleGraphData(stock, event) {
        this.displayError = false;
        stock.selected = !stock.selected;

        // verify that maximum 6 stocks can be selected
        (stock.selected) ? this.selectedCounter++ : this.selectedCounter--;
        if (this.selectedCounter > 6) {
          stock.selected = false;
          this.selectedCounter--;
          this.error = 'Max 6 stocks allowed'
          this.displayError = true;
        } else this.drawGraph();
      },


      drawGraph() {
        //convert data to highcharts-data
        let graphData = [];
        this.stocksData.forEach(stock => {
          if (stock.selected) {
            let {ChangeFromYearLowPercent, ChangeFromYearHighPercent, symbol} = stock;
            graphData.push({ data: [ChangeFromYearLowPercent, ChangeFromYearHighPercent], name: symbol })
          }
        });

        // INIT CHART THEME + DRAW IT

        Highcharts.theme = {
          colors: ['#2b908f', '#90ee7e', '#f45b5b', '#7798BF', '#aaeeee', '#ff0066', '#eeaaee',
            '#55BF3B', '#DF5353', '#7798BF', '#aaeeee'],
          chart: {
            backgroundColor: {
              linearGradient: { x1: 0, y1: 0, x2: 1, y2: 1 },
              stops: [
                [0, '#727274'],
                [1, '#323234']
              ]
            },
            style: {
              fontFamily: 'Merriweather, sans-serif'
            },
            plotBorderColor: '#606063'
          },
          title: {
            style: {
              color: '#E0E0E3',
              textTransform: 'uppercase',
              fontSize: '20px'
            }
          },
          subtitle: {
            style: {
              color: '#E0E0E3',
              textTransform: 'uppercase'
            }
          },
          xAxis: {
            gridLineColor: '#707073',
            labels: {
              style: {
                color: '#E0E0E3'
              }
            },
            lineColor: '#707073',
            minorGridLineColor: '#505053',
            tickColor: '#707073',
            title: {
              style: {
                color: '#A0A0A3'

              }
            }
          },
          yAxis: {
            gridLineColor: '#707073',
            labels: {
              style: {
                color: '#E0E0E3'
              }
            },
            lineColor: '#707073',
            minorGridLineColor: '#505053',
            tickColor: '#707073',
            tickWidth: 1,
            title: {
              style: {
                color: '#A0A0A3'
              }
            }
          },
          tooltip: {
            backgroundColor: 'rgba(60, 60, 60, 0.85)',
            style: {
              color: '#F0F0F0'
            }
          },
          plotOptions: {
            series: {
              dataLabels: {
                color: '#B0B0B3'
              },
              marker: {
                lineColor: '#333'
              }
            },
            boxplot: {
              fillColor: '#505053'
            },
            candlestick: {
              lineColor: 'white'
            },
            errorbar: {
              color: 'white'
            }
          },
          legend: {
            itemStyle: {
              color: '#E0E0E3'
            },
            itemHoverStyle: {
              color: '#FFF'
            },
            itemHiddenStyle: {
              color: '#606063'
            }
          },
          credits: {
            style: {
              color: '#666'
            }
          },
          labels: {
            style: {
              color: '#707073'
            }
          },

          drilldown: {
            activeAxisLabelStyle: {
              color: '#F0F0F3'
            },
            activeDataLabelStyle: {
              color: '#F0F0F3'
            }
          },

          navigation: {
            buttonOptions: {
              symbolStroke: '#DDDDDD',
              theme: {
                fill: '#505053'
              }
            }
          },

          // scroll charts
          rangeSelector: {
            buttonTheme: {
              fill: '#505053',
              stroke: '#000000',
              style: {
                color: '#CCC'
              },
              states: {
                hover: {
                  fill: '#707073',
                  stroke: '#000000',
                  style: {
                    color: 'white'
                  }
                },
                select: {
                  fill: '#000003',
                  stroke: '#000000',
                  style: {
                    color: 'white'
                  }
                }
              }
            },
            inputBoxBorderColor: '#505053',
            inputStyle: {
              backgroundColor: '#333',
              color: 'silver'
            },
            labelStyle: {
              color: 'silver'
            }
          },

          navigator: {
            handles: {
              backgroundColor: '#666',
              borderColor: '#AAA'
            },
            outlineColor: '#CCC',
            maskFill: 'rgba(255,255,255,0.1)',
            series: {
              color: '#7798BF',
              lineColor: '#A6C7ED'
            },
            xAxis: {
              gridLineColor: '#505053'
            }
          },

          scrollbar: {
            barBackgroundColor: '#808083',
            barBorderColor: '#808083',
            buttonArrowColor: '#CCC',
            buttonBackgroundColor: '#606063',
            buttonBorderColor: '#606063',
            rifleColor: '#FFF',
            trackBackgroundColor: '#404043',
            trackBorderColor: '#404043'
          },

          legendBackgroundColor: 'rgba(0, 0, 0, 0.5)',
          background2: '#505053',
          dataLabelsColor: '#B0B0B3',
          textColor: '#C0C0C0',
          contrastTextColor: '#F0F0F3',
          maskColor: 'rgba(255,255,255,0.3)'
        };

        // Apply the theme
        Highcharts.setOptions(Highcharts.theme);

        Highcharts.chart('graph-container', {
          chart: {
            type: 'column'
          },
          title: {
            text: 'Difference in Percentage of Year Low and Year High'
          },
          xAxis: {
            categories: ['YearLowOffset', 'YearHighOffset']
          },
          credits: {
            enabled: true
          },
          series: graphData
        });
      }
    },
  }

</script>

<style scoped lang="scss">
  @import url('//maxcdn.bootstrapcdn.com/font-awesome/4.2.0/css/font-awesome.min.css');

  * {
    box-sizing: border-box;
  }
  
  .data-display-container {
    display: flex;
    flex-wrap: nowrap;
    justify-content: space-between;
    padding-left: 15px;
    padding-right: 15px;
  }
  
  .table-container {
    flex: 0 1 45%;
  }
  
  .graph-container {
    flex: 0 1 45%;
    max-height: 450px;
  }
  
  .header-container {
    padding-left: 15px;
    padding-right: 15px;
    padding-bottom: 3px;
    margin-bottom: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .search-container {
    display: flex;
    height: 35px;
    justify-content: flex-start;
    flex-grow: 1;
  }
  
  $button-bg-color: #348AA7;
  $button-border-color: #166E84;

  button {
    margin-right: 5px;
    height: 92%;
    min-width: 100px;
    background: $button-bg-color;
    border: none;
    outline: none;
    color: white;
    font-family: inherit;
    font-weight: 400;
    font-size: 16px;
    border-radius: 3px;
    box-shadow: 0 5px 0px darken($button-bg-color, 5%);
    cursor: pointer;
    &:hover {
      background: darken($button-bg-color, 5%);
      box-shadow: 0 4px 1px darken($button-bg-color, 8%);
      transition: all 0.1s ease-in;
    }
    &:active {
      transform: translateY(4px);
      border-bottom-width: 2px;
      box-shadow: none;
    }
  }
  
  input {
    font-size: 18px;
    height: 100%;
    border: 0;
    font-family: Merriweather;
    margin-right: 15px;
    border-bottom: 1px solid #ccc;
    -webkit-appearance: none;
    border-radius: 3px;
    padding: 5px 0;
    &:focus {
      outline: 0;
      border-color: coral;
    }
  }
  
  .error-msg {
    padding: 10px;
    border-radius: 3px 3px 3px 3px;
    color: #D8000C;
    font-weight: 100;
    font-size: 14px;
  }
</style>

<style src="./table.scss" scoped lang="scss"></style>
<style src="./transitions.scss" scoped lang="scss"></style>