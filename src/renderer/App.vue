<template>
  <div id="app">
    <modals-container />
    <transition name="fade" mode="out-in">
      <router-view></router-view>
    </transition>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'
  import worker from './workers/promise'
  import Constant from '../Constant'
  import CryptoJS from 'crypto-js'

  // const { NFC } = require('nfc-pcsc')

  export default {
    name: 'uimd_pb_ia',
    computed: {
      ...mapGetters([
        Constant.GET_CURRENT_USER,
        Constant.GET_IS_SHOW_MODAL_BARCODE,
        Constant.GET_BACKEND_DATA,
        // lis 단축키 추가
        Constant.GET_SETTINGS,
      ])
    },
    data () {
      return {
        currentReader: null,
        barcodeStr: null,
        backendData: null,
        // lis 단축키 추가
        lisHotKey: '',
      }
    },
    watch: {
      // lis 단축키 추가
      'getSettings.lisHotKey': function (newLisHotKey, oldLisHotKey) {
        // 기존 단축키 제거
        this.removeLisHotKey();

        //새로운 단축키 등록
        this.setupLisHotKey(newLisHotKey);

        console.log('lis 단축키')
        console.log(newLisHotKey, oldLisHotKey)
      }
    },
    mounted () {
      var self = this

      // lis 단축키 추가
      this.setupLisHotKey(this.getSettings.lisHotKey && this.getSettings.lisHotKey)
      // console.log('store에 저장되어 있는 lis hot key 확인')
      // console.log(this.getSettings.lisHotKey)


      self.backendData = self.getBackendData

      console.log('백엔드데이터')
      console.log(self.backendData)

      console.log('user!!!')
      console.log(this.getCurrentUser.userId)
      // 만약 현재 유저의 userId가 존재하지 않는다면, localStorage에 "getcurrentuser"를 저장합니다.
      if (!this.getCurrentUser.userId) {
          // "getcurrentuser"를 localStorage에 저장합니다.
          localStorage.setItem('memoTitle', title);
          console.log('localStorage에 "getcurrentuser" 저장됨');
      }

      window.addEventListener('keyup', function(event) {
        // console.log(event)
        // console.log(self.$route.path)


        // 예를 들어 WBC에서 키를 누르면 (keyup)
        // self.$route.path.indexOf("wbcReport")
        // 60 (해당 되는 페이지에서 양수가 나옴 왜 60인지는)
        // self.$route.path.indexOf("rbcReport")
        // -1
        // self.$route.path.indexOf("finalReport")
        // -1


        // if (self.$route.path.indexOf('wbcReport') > 0 ||
        //     self.$route.path.indexOf('rbcReport') > 0 ||
        //     self.$route.path.indexOf('finalReport') > 0) {
        //     // => Database 탭에서 리스트 더블클릭해서 상세 페이지 안으로 들어온 상태
        //   if (event.key === 'F9') {
        //     self.EventBus.$emit('UPLOAD_LIS', null)
        //   } else if (event.key === 'Enter') {
        //     // self.EventBus.$emit('CLICK_ENTER', null)
        //   }
        // }

        // resultpage라면
        // if (event.key === 'F9') {
        //   self.EventBus.$on('받아서 보낼거')
        // }

        if (event.key === 'Escape') {
          // off magnifyingGlass
          self.EventBus.$emit('SET_MAGNIFY', false)

          // off ruler
          self.EventBus.$emit('SET_RULER', {id: 'none'})
        }
      })

      // Home.vue의 공통처리로 넘김
      
      // console.log('[APP] self.backendData : ' + JSON.stringify(self.backendData))
      // if (self.backendData && self.backendData.jobCmd === 'BARCODE_REG') {
      //   if (self.backendData.resultCd === '00000') {
      //     self.$toasted.show(Constant.IDS_BARCODE_REGISTRATION_SUCCESSFUL, {
      //       position: 'bottom-center',
      //       duration: '2000'
      //     })
      //   }
      // }

      // this.$nextTick(function() {
      //   console.log('load rfid---')
      //   try {
      //     self.nfc = new NFC()

      //     self.nfc.on('reader', function (reader) {
      //       // tag card
      //       reader.on('card', function (card) {
      //         console.log(self.$router.currentRoute.path)
      //         console.log(reader.reader.name + 'card inserted', card)
      //         console.log(self.getIsShowModalBarcode)

      //         // 바코드 등록 modal 있을때만 등록함
      //         if (self.getIsShowModalBarcode) {
      //           const CLASSIC_1K = '000100000000'
      //           const CLASSIC_4K = '000200000000'
      //           const ULTRALIGHT = '000300000000'
      //           const KEY_A = 'FFFFFFFFFFFF'
      //           const KEY_TYOE_A = 0x60

      //           let buf = card.atr
      //           let type = buf.slice(0,12).toString('hex').toUpperCase()
      //           let version = null

      //           if (type == '3B8F8001804F0CA000000306') {
      //             version = card.atr.slice(13,19).toString('hex')
      //             switch (version) {
      //               case '000100000000':
      //                 console.log('Mifare Classic 1k')
      //                 break;
      //               case '000200000000':
      //                 console.log('Mifare Classic 4k')
      //                 break;
      //               case '000300000000':
      //                 console.log('Mifare Ultralight')
      //                 break;
      //               default:
      //                 console.log('Other card')
      //             }
      //           }

      //           console.log(version)

      //           if (version === '000100000000') {
      //             reader.authenticate(4, KEY_TYOE_A, KEY_A).then(function(isAuth) {
      //               console.log(isAuth)
      //               if (isAuth) {
      //                 reader.read(4, 48, 16).then(function(data) {
      //                   console.log(data)
      //                   self.barcodeStr = data.toString('hex', 0, 44)
      //                   self.currentReader = reader

      //                   console.log(self.barcodeStr)
      //                   // console.log(self.$hexToString(self.barcodeStr))
      //                   // ***** self.barcodeStr 복호화 및 수량 조회 에러 처리 필요 *****/
      //                   if (self.barcodeStr.replace(/[^(1-9)]/gi, '') === '') {
      //                     self.$toasted.show(Constant.IDS_THIS_CARD_HAS_ALREADY_BEEN_USED, {
      //                       position: 'bottom-center',
      //                       duration: '2000'
      //                     })

      //                     return
      //                   }

      //                   let key = CryptoJS.enc.Utf8.parse(Constant.REMEMBER_PWD_KEY)
      //                   let iv = CryptoJS.enc.Utf8.parse(Constant.REMEMBER_PWD_IV)

      //                   let decBarcode = CryptoJS.AES.decrypt(self.barcodeStr.substr(0, 64), key, {
      //                     iv: iv,
      //                     mode: CryptoJS.mode.CBC,
      //                     format: CryptoJS.format.Hex
      //                   }).toString(CryptoJS.enc.Utf8)
      //                   console.log(decBarcode)

      //                   if (decBarcode !== '') {
      //                     const writeBuf = Buffer.allocUnsafe(48)
      //                     writeBuf.fill(0)

      //                     reader.write(4, writeBuf, 16).then(function(succ) {
      //                       console.log(succ)
      //                       self.sendBarcodeReg(decBarcode)
      //                       // self.$toasted.show(Constant.IDS_BARCODE_REGISTRATION_SUCCESSFUL, {
      //                       //   position: 'bottom-center',
      //                       //   duration: '2000'
      //                       // })
      //                     }, function(err) {
      //                       console.log(err)
      //                       self.$toasted.show(err, {
      //                         position: 'bottom-center',
      //                         duration: '2000'
      //                       })
      //                     })
      //                   } else {
      //                     self.$toasted.show(Constant.IDS_THIS_CARD_HAS_ALREADY_BEEN_USED, {
      //                       position: 'bottom-center',
      //                       duration: '2000'
      //                     })
      //                   }

      //                 }, function (err) {
      //                   console.log('error when reading data', err)
      //                   self.$toasted.show(err, {
      //                     position: 'bottom-center',
      //                     duration: '2000'
      //                   })
      //                 })
      //               }
      //             }, function (err) {
      //               console.log('authenticate error', err)
      //               self.$toasted.show(err, {
      //                 position: 'bottom-center',
      //                 duration: '2000'
      //               })
      //             })
      //           }
      //         }
      //       })

      //       reader.on('card.off', function (card) {
      //         console.log(reader.reader.name + 'card remove' + card)
      //       })

      //       reader.on('error', function (err) {
      //         console.log(reader.reader.name + 'an error occurred' + err)
      //       })

      //       reader.on('end', function () {
      //         console.log(reader.reader.name + 'device removed')
      //       })
      //     })

      //     self.nfc.on('error', function (err) {
      //       console.log('an error occurred' + err)
      //     })
      //   } catch (err) {
      //     console.log(err.message)
      //   }
      // })
    },
    methods: {
      // lis 단축키 추가
      setupLisHotKey (lisHotKey) {

        var self = this;

        this.keyupListener = function (event) {
          if (self.$route.path.indexOf('wbcReport') > 0 ||
              self.$route.path.indexOf('rbcReport') > 0 ||
              self.$route.path.indexOf('finalReport') > 0) {
              if (event.key === lisHotKey) {
                  self.EventBus.$emit('UPLOAD_LIS', null);
              }
          }
        }

        window.addEventListener('keyup', this.keyupListener)

      },
      // 이전 lis hot key 제거
      removeLisHotKey () {
        if (this.keyupListener) {
          window.removeEventListener('keyup', this.keyupListener)
        }
      },
      goMain (event) {
        this.$router.push('/')
      },
      sendBarcodeReg (barcodeNo) {
        console.log(barcodeNo)

        var deviceBarcodeNo = barcodeNo.substr(0, 12)
        var type = barcodeNo.substr(12, 2)
        var count = barcodeNo.substr(14, 10)

        console.log(deviceBarcodeNo)
        console.log(type)
        console.log(count)

        // 바코드 regist
        var params = {
          jobCmd: 'BARCODE_REG',
          deviceBarcodeNo: deviceBarcodeNo,
          type: type,
          count: count,
          reqUserId: this.getCurrentUser.userId,
          reqDttm: this.$getDateTimeStr()
        }

        worker.sendDataToServer(this, JSON.stringify(params))
      }
    }
  }
</script>

<style>
  .fade-enter-active {
    transition: all .3s ease;
  }
  .fade-leave-active {
    transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>
