<!DOCTYPE html>
<html>
  <head>
    <title>Pancake Analytics Dashboard</title>
    <link
      rel="stylesheet"
      media="all"
      href="https://cdn.jsdelivr.net/gh/bitquery/widgets@v1.3.8/dist/assets/css/widgets.css"
    />
    <link
      rel="stylesheet"
      href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
      integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
      crossorigin="anonymous"
    />
    <script src="https://cdn.jsdelivr.net/gh/bitquery/widgets@v1.3.8/dist/widgets.js"></script>
    <script
      src="https://code.jquery.com/jquery-3.2.1.slim.min.js"
      integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN"
      crossorigin="anonymous"
    ></script>
    <script
      src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js"
      integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q"
      crossorigin="anonymous"
    ></script>
    <script
      src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js"
      integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl"
      crossorigin="anonymous"
    ></script>
    <style></style>
  </head>
  <body>
    <div class="container">
      <div class="row" style="margin-top:20px; margin-bottom:20px">
        <div class="col">
          <div class="d-flex justify-content-center" style="font-weight: bold;">
            Pancake Analytics Dashboard
          </div>
        </div>
      </div>
      <div class="row" style="margin-top:20px; margin-bottom:20px">
        <div class="col-md-6">
          <div class="d-flex justify-content-center" style="font-weight: bold;">
            Daily Active Address 
          </div>
          <div id="active_address_by_time"></div>
        </div>
        <div class="col-md-6">
          <div class="d-flex justify-content-center" style="font-weight: bold;">
            Daily LP Pool Liquidity Activity
          </div>

          <div id="lp_liquidity_activity"></div>
        </div>
      </div>
      <div class="row" style="margin-top:20px; margin-bottom:20px">
        <div class="col-md-6">
          <div class="d-flex justify-content-center" style="font-weight: bold;">
            DexTrades GAS Fee Contribution (BNB)
          </div>

          <div id="gas_fee_contribution_by_exchange"></div>
        </div>
        <div class="col-md-6">
          <div class="d-flex justify-content-center" style="font-weight: bold;">
            Gas Price By Date ( Median, Average, Maximum) in Log scale, GWei
          </div>

          <div id="transaction_gas_price_by_time_log"></div>
        </div>
      </div>
      <div class="row" style="margin-top:20px; margin-bottom:20px">
        <div class="col">
          <div class="d-flex justify-content-end">
            <a href="https://explorer.bitquery.io/bsc/smart_contract/0xbcfccbde45ce874adcb698cc183debcf17952812">
              <img
                src="https://bitquery.io/wp-content/uploads/2020/09/bitquery_logo_w.png"
                style="height: 30px;"
            /></a>
          </div>
        </div>
      </div>
    </div>
  </body>

  <script>
    (function() {
      widgets.init("https://graphql.bitquery.io", "", {
        locale: "en",
        theme: "light",
      });
      var query = new widgets.query(`
      query ($dateFormat: String!)  
      {ethereum(network:bsc){
  dexTrades(date:{since:"2020-09-24"}){
  Pancake:count(uniq:senders,exchangeName:{is:"Pancake"})
  Others:count(uniq:senders,exchangeName:{not:"Pancake"})
    date:date{dt:date(format:$dateFormat)}
   gasValue
  
  }
}}`);
      var wdts = new widgets.chartByTime(
        "#active_address_by_time",
        query,
        "ethereum.dexTrades",
        {
          title: "Dynamic Active Address",
          chartOptions: {
            vAxes: {
              "0": {
                title: "Daily Active Address",
                scaleType:"linear"
              },
            },
            series: {
              "0": {
                color: "#ffc107",
              },
              "1":{
                color:"#28a745"
              }
            },
            seriesType: "lines",
          },
          dataOptions: [
          {
              title: {
                label: "Date",
                type: "date",
              },
              path: "date.dt",
            },
            {
              title: "Pancake",
              path: "Pancake"
            },
            {
              title: "Others",
              path: "Others"
            },
          ],
        }
      );
      query.request({
        dateFormat:"%Y-%m-%d"
      });
    })();
  </script>
  <script>
    (function() {
      widgets.init("https://graphql.bitquery.io", "", {
        locale: "en",
        theme: "light",
      });
      var query = new widgets.query(`
      query ($address: [String!],
      $dateFormat: String!) {
  ethereum(network: bsc) {
    smartContractEvents(
      smartContractAddress: {in: $address}
      smartContractEvent: {in: ["Mint","Burn"]}
      date:{since:"2020-11-01"}
    ) {
      date:date{date(format:$dateFormat)}
   Mint:count(uniq: callers,smartContractEvent:{is:"Mint"})
 Burn:count(uniq: callers,smartContractEvent:{is:"Burn"})
      
    }
  }
}



`);
      var wdts = new widgets.chartByTime(
        "#lp_liquidity_activity",
        query,
        "ethereum.smartContractEvents",
        {
          title: "LP Pool Liquidity Activity",
          chartOptions: {
            vAxes: {
              "0": {
                title: "Liquidity Activity",
              },
            },
            series: {
              "0": {
                color: "#ffc107",
              },
              "1":{
                color:"28a745",
              }
            },
            seriesType: "area"
          },
          dataOptions: [
          {
              title: {
                label: "Date",
                type: "date",
              },
              path: "date.date",
            },
            {
              title: "Liquidity Adder",
              path: "Mint"
            },
            {
              title: "Liquidity Remover",
              path: "Burn"
            },
          ],
        }
      );
      query.request({
        address:[
"0xA527a61703D82139F8a06Bc30097cC9CAA2df5A6",
"0x7b1e440240B220244761C9D9A3B07fbA1995BD84",
"0x7Bb89460599Dbf32ee3Aa50798BBcEae2A5F7f6a",
"0xe022baa3E5E87658f789c9132B10d7425Fd3a389",
"0xfEc200A5E3adDD4a7915a556DDe3F5850e644020",
"0x0F556f4E47513d1a19Be456a9aF778d7e1A226B9",
"0xCA01F5D89d5b1d24ca5D6Ecc856D21e8a61DAFCc",
"0xad7e515409e5a7d516411a85acc88c5e993f570a",
"0x83B92D283cd279fF2e057BD86a95BdEfffED6faa",
"0xbF36959939982D0D34B37Fb3f3425da9676C13a3",
"0x45a9e8d48bc560416008d122c9437927fed50e7d",
"0x69ab367bc1bea1d2c0fb4dbaec6b7197951da56c",
"0xAB97952a2806D5c92b7046c7aB13a72A87e0097b",
"0x4db28767d1527ba545ca5bbda1c96a94ed6ff242",
"0xdc6c130299e53acd2cc2d291fa10552ca2198a6b",
"0x292ca56ed5b3330a19037f835af4a9c0098ea6fa",
"0x4D5aA94Ce6BbB1BC4eb73207a5a5d4D052cFcD67",
"0x36b8b28d37f93372188f2aa2507b68a5cd8b2664",
"0x86e650350c40a5178a5d014ba37fe8556232b898",
"0x9d8b7e4a9d53654d82f12c83448d8f92732bc761",
"0x17580340f3daedae871a8c21d15911742ec79e0f",
"0x9e642d174b14faea31d842dc83037c42b53236e6",
"0x4576C456AF93a37a096235e5d83f812AC9aeD027",
"0x5E3CD27F36932Bc0314aC4e2510585798C34a2fC",
"0xb5ab3996808c7e489dcdc0f1af2ab212ae0059af",
"0xc1800c29cf91954357cd0bf3f0accaada3d0109c",
"0x0392957571f28037607c14832d16f8b653edd472",
"0xcbe2cf3bd012e9c1ade2ee4d41db3dac763e78f3",
"0x99d865ed50d2c32c1493896810fa386c1ce81d91",
"0xeb325a8ea1c5abf40c7ceaf645596c1f943d0948",
"0x60bB03D1010b99CEAdD0dd209b64bC8bd83da161",
"0x66b9e1eac8a81f3752f7f3a5e95de460688a17ee",
"0x74690f829fec83ea424ee1f1654041b2491a7be9",
"0x3ef4952c7a9afbe374ea02d1bf5ed5a0015b7716",
"0xD1F12370b2ba1C79838337648F820a87eDF5e1e6",
"0xc92Dc34665c8a21f98E1E38474580b61b4f3e1b9",
"0x852A68181f789AE6d1Da3dF101740a59A071004f",
"0xF609ade3846981825776068a8eD7746470029D1f",
"0xD5664D2d15cdffD597515f1c0D945c6c1D3Bf85B",
"0xffb9e2d5ce4378f1a89b29bf53f80804cc078102",
"0x36b7d2e5c7877392fb17f9219efad56f3d794700",
"0x6411310c07d8c48730172146fd6f31fa84034a8b",
"0x91589786D36fEe5B27A5539CfE638a5fc9834665",
"0xbc765fd113c5bdb2ebc25f711191b56bb8690aec",
"0x680dd100e4b394bda26a59dd5c119a391e747d18",
"0x3aB77e40340AB084c3e23Be8e5A6f7afed9D41DC",
"0x20781bc3701c5309ac75291f5d09bdc23d7b7fa8",
"0x01ecc44ddd2d104f44d2aa1a2bd9dfbc91ae8275",
"0xbe14f3a89a4f7f279af9d99554cf12e8c29db921",
"0x64373608f2e93ea97ad4d8ca2cce6b2575db2f55",
"0xd6b900d5308356317299dafe303e661271aa12f1",
"0xd5b3ebf1a85d32c73a82b40f75e1cd929caf4029",
"0x58B58cab6C5cF158f63A2390b817710826d116D0",
"0x470bc451810b312bbb1256f96b0895d95ea659b1",
"0x51a2ffa5b7de506f9a22549e48b33f6cf0d9030e",
"0x9c4f6a5050cf863e67a402e8b377973b4e3372c1",
"0xbEA35584b9a88107102ABEf0BDeE2c4FaE5D8c31",
"0xff17ff314925dff772b71abdff2782bc913b3575",
"0xC743Dc05F03D25E1aF8eC5F8228f4BD25513c8d0",
"0x9f40e8a2fcaa267a0c374b6c661e0b372264cc3d",
"0x1b96b92314c44b159149f7e0303511fb2fc4774f",
"0xba51d1ab95756ca4eab8737ecd450cd8f05384cf",
"0xc639187ef82271d8f517de6feae4faf5b517533c",
"0xbcd62661a6b1ded703585d3af7d7649ef4dcdb5c",
"0x981d2ba1b298888408d342c39c2ab92e8991691e",
"0xaebe45e3a03b734c68e5557ae04bfc76917b4686",
"0xc15fa3E22c912A276550F3E5FE3b0Deb87B55aCd",
"0x610e7a287c27dfFcaC0F0a94f547Cc1B770cF483",
"0x41182c32F854dd97bA0e0B1816022e0aCB2fc0bb",
"0x70D8929d04b60Af4fb9B58713eBcf18765aDE422",
"0x7561EEe90e24F3b348E1087A005F78B4c8453524",
"0x4e0f3385d932F7179DeE045369286FFa6B03d887",
"0x20bcc3b8a0091ddac2d0bc30f68e6cbb97de59cd",
"0xc7b4b32a3be2cb6572a1c9959401f832ce47a6d2",
"0x2333c77fc0b2875c11409cdcd3c75d42d402e834",
"0x574a978c2d0d36d707a05e459466c7a1054f1210",
"0x56c77d59e82f33c712f919d09fceddf49660a829",
"0x5acac332f0f49c8badc7afd0134ad19d3db972e6",
"0x54edd846db17f43b6e43296134ecd96284671e81",
"0x68Ff2ca47D27db5Ac0b5c46587645835dD51D3C1",
"0x4269e7F43A63CEA1aD7707Be565a94a9189967E9",
"0x35fe9787f0ebf2a200bac413d3030cf62d312774",
"0x7a34bd64d18e44CfdE3ef4B81b87BAf3EB3315B6",
"0x30479874f9320a62bce3bc0e315c920e1d73e278",
"0x752E713fB70E3FA1Ac08bCF34485F14A986956c4",
"0x7793870484647a7278907498ec504879d6971EAb",
"0x7cd05f8b960ba071fdf69c750c0e5a57c8366500",
"0x745c4fd226e169d6da959283275a8e0ecdd7f312",
"0x2730bf486d658838464a4ef077880998d944252d",
"0x970858016C963b780E06f7DCfdEf8e809919BcE8",
"0xc2eed0f5a0dc28cfa895084bc0a9b8b8279ae492",
"0xd937FB9E6e47F3805981453BFB277a49FFfE04D7",
"0x3Da30727ed0626b78C212e81B37B97A8eF8A25bB"],
dateFormat:"%Y-%m-%d"

      });
    })();
  </script>
  <script>
    (function() {
      widgets.init("https://graphql.bitquery.io", "", {
        locale: "en",
        theme: "light",
      });
      var query = new widgets.query(`
      query ($dateFormat:String!){
  ethereum(network:bsc) {
    dexTrades(
    date: {after: "2020-12-01"}) {
      
			date:date{date(format:$dateFormat)}
      
      exchange {
        fullName
      }
      gasValue

 
    }
  }
}`);
var wdts = new widgets.pivotChart('#gas_fee_contribution_by_exchange', query, 'ethereum.dexTrades', {
 "title": "<span class=\"translation_missing\" title=\"translation missing: en.widgets.titles.gas_fee_contribution_by_exchange\">DexTrades GAS Fee Contributio>",
 "chartOptions": {
  "vAxes": {
   "0": {
    "title": "Trades Count",
    "viewWindow": {
     "min": 0
    }
   }
  },
 
  "bar": {
   "groupWidth": "100%"
  },
  "isStacked": true
 },
 "cols": [
  {
   "title": "Exchange",
   "path": "exchange.fullName",
   "metrics": [
    {
     "title": "Gas",
     "path": "gasValue"
    }
   ]
  }
 ],
 "rows": [
  {
   "title": "Date",
   "path": "date.date"
  }
 ],
 "sort": {
  "metric": "Gas",
  "direction": "asc",
  "axis": "col"
 },
 "limit": {}
});
    query.request({"dateFormat": "%Y-%m-%d"});
    })()
</script>
  <script>
    (function() {
      widgets.init("https://graphql.bitquery.io", "", {
        locale: "en",
        theme: "light",
      });
      var query = new widgets.query(`
            query ($network: EthereumNetwork!,
                  $dateFormat: String!){
                    ethereum(network: $network ){
                      transactions(options:{asc: "date.date"},
                      date:{since:"2020-09-14"}) {
                        date: date{
                          date(format: $dateFormat)
                        }
                        gasPrice
                        gasValue
                        average: gasValue(calculate: average )
                        maxGasPrice: gasPrice(calculate: maximum)
                        medianGasPrice: gasPrice(calculate: median)
                      }
                    }
                  }`);
      var wdts = new widgets.chartByTime(
        "#transaction_gas_price_by_time_log",
        query,
        "ethereum.transactions",
        {
          title:
            "Gas Price By Date ( Median, Average, Maximum) in Log scale, GWei",
          chartOptions: {
            vAxes: {
              "0": {
                title: "Avg. Gas Price, GWei",
                scaleType: "log",
              },
            },
            series: {
              "0": {
                color: "#28a745",
              },
              "1": {
                color: "#ffc107",
              },
              "2": {
                color: "#9bc2cf",
              },
            },
            seriesType: "lines",
          },
          dataOptions: [
            {
              title: {
                label: "Date",
                type: "date",
              },
              path: "date.date",
            },
            {
              title: "Avg. Gas Price, GWei",
              path: "gasPrice",
            },
            {
              title: "Maximum Gas Price, GWei",
              path: "maxGasPrice",
            },
            {
              title: "Median Gas Price, GWei",
              path: "medianGasPrice",
            },
          ],
        }
      );
      query.request({
        network: "bsc",
        dateFormat: "%Y-%m-%d",
      });
    })();
  </script>
</html>
