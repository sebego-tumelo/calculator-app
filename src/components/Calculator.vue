<template>
 <div class="my-calculator">
    <div class="my-display ">
        <div>{{current || 0}}</div>
        <div>{{previous}}</div>
    </div>
  <div @click="clear" class="btn">C</div>
  <div @click="sign" class="btn">-/+</div>
  <div @click="append('%')" class="btn">%</div>
  <div @click="append('/')" class="btn operator">/</div>
  <div @click="append('7')" class="btn ">7</div>
  <div @click="append('8')" class="btn">8</div>
  <div @click="append('9')" class="btn">9</div>
  <div @click="append('x')" class="btn operator">x</div>
  <div @click="append('4')" class="btn">4</div>
  <div @click="append('5')" class="btn">5</div>
  <div @click="append('6')" class="btn">6</div>
  <div @click="append('-')" class="btn operator">-</div>
  <div @click="append('1')" class="btn">1</div>
  <div @click="append('2')" class="btn">2</div>
  <div @click="append('3')" class="btn">3</div>
  <div @click="append('+')" class="btn operator">+</div>
  <div @click="append('0')" class="zero btn">0</div>
  <div @click="dot" class="btn">.</div>
  <div @click="equal" class="btn operator">=</div>
  
 </div>
</template>

<script>


export default {
    data(){
        return{
            current: '',
            operators : ['x', '/', '+', '-', '#' ],
            store: [{A:''},{B:''}, {C:''}, {D:''}, {E:''}, {F:''}],
            previous: '',
            lockOperand : {operand: '', operator: '', locked: false},
            opPointer : 0
        }
    },
    methods:{
        clear(){
            this.current = ''
        },
        sign(){
            this.current = this.current.charAt(0) === '-' ? this.current.slice(1) : `-${this.current}`
        },
        percent(){
            this.current = parseFloat(this.current)/100
        },
        append(number){
            if(this.operatorClicked){
                // this.current = ''
                this.operatorClicked = false
            }
            this.current = `${this.current}${number}`
        },
        dot(){
            if(this.current.indexOf('.') === -1){
                this.append('.')
            }
        },
       
       
        evaluate(obj){

           

            let result = ''

            // is the current operator contained in current
            this.operators.every( op => {
                
                if(this.current.includes(op)){

                    return false
                }else{
                    
                    this.opPointer++
                }
                return true
            });

            console.log('opPointer: ' + this.opPointer)
            console.log('currentOp: ' + obj.currentOp)
            console.log('selected Op: ' + this.operators[this.opPointer])

            if(obj.currentOp != this.operators[this.opPointer]){

                return -1
            }else{
                switch (this.opPointer) {
                    case 0:
                        result = obj.a * obj.b                        
                        break;
                    case 1:
                        result = obj.a / obj.b                        
                        break;
                    case 2:
                        result = obj.a + obj.b     
                        break;
                    case 3:
                        result = obj.a - obj.b                        
                        break;    
                    default:
                        break;
                }

                this.current = this.current.replaceBetween(obj.startIndex, obj.endIndex+1, result)

                return 1
            }
        },

        encodeInput(){

           

            // is lockOperand.lock true
            if(this.lockOperand.locked){

                // append current 
                this.current = this.current + this.lockOperand.operator + this.lockOperand.operand
            }

            this.current = this.current + '#'

            this.store.forEach(el => {

                let n = Object.keys(el)[0]
                if(this.current.includes(n)){

                    this.current = this.current.replace(/n/g, el[n])
                }
            })

            // check for percentage symbols
            if(this.current.includes('%')){

                let count = this.current.split('%').length

                for(let i = 0; i < count ; i++){

                    let endPos = this.current.indexOf('%')
                    let pointer = endPos - 1
                    let number = ''
                    let startPos = 0

                    while(pointer >= 0){

                        let char = this.operators.find(this.current.charAt(pointer))
                        startPos = pointer
                        
                        if(char){

                            // an operator
                            break
                        }else{
                            number = number + char
                        }

                        --pointer
                    }

                    number = parseFloat(number)/100

                    this.current.replaceBetween(startPos, endPos+1, numbers)

                    
                }
            }

        }, 

        decodeInput(){

            this.current = this.current.replace('#', '')

            this.operators.forEach(el => {
                
                if(this.current.endsWith(el)){
                    
                    this.current = this.current.slice(this.current.lastIndexOf(el))
                }
            })
        }, 

        

        equal(){

            String.prototype.replaceBetween = function(start, end, what) {
            return this.substring(0, start) + what + this.substring(end);
            };
 

            let a = ''
            let b = ''
            let opIndex = 0
            let startIndex = 0
            let endIndex = 0
            let operatorChecked = false
            let currentOp = ''
          
            this.previous = this.current
            this.encodeInput()
            
            let endloop = false

            while(endloop == false){

                for(let x = 0; x < this.current.length; x++){

                    let n = this.current.charAt(x)
                    
                    

                    if(!this.operators.includes(n)){

                        // a number
                        // is operatorCheck true
                        operatorChecked? b = b + n: a = a + n
                        endIndex++

                    }else{

                        // an operator
                        operatorChecked = true

                        if(!(b == '')){

                            if(this.evaluate({a:parseFloat(a), b:parseFloat(b), currentOp: currentOp, startIndex: startIndex, endIndex:endIndex -1  }) == -1){

                                a = b
                                b = ''
                                currentOp = n
                                startIndex = opIndex + 1
                                opIndex = endIndex
                                endIndex++
                            }else{

                                 a = ''
                                 b = ''
                                 startIndex = 0
                                 endIndex = 0
                                 opIndex = 0
                                 operatorChecked = false
                                 break
                            }
                        }else{

                            if(this.current.charAt(x+1) == '#'){

                                this.lockOperand.locked = true
                                this.lockOperand.operand = a
                                this.lockOperand.operator = n

                                this.decodeInput()
                                endloop = true
                                break
                            }

                            if(n != '#'){

                                currentOp = n
                                opIndex = endIndex + 1
                                endIndex++
                            }else{

                                if(!this.lockOperand.locked) this.opPointer = 0 
                                
                                a = ''
                                 b = ''
                                 startIndex = 0
                                 endIndex = 0
                                 opIndex = 0
                                 operatorChecked = false

                                this.decodeInput()
                                endloop = true
                                break
                            }
                        }

                    }

                    console.log('n: '+n)
                    console.log('index: '+endIndex)
                    console.log('a: '+a)
                    console.log('b: '+b)
                    console.log('current Op: '+currentOp)
                    console.log('start Index: '+startIndex)
                    console.log('end Index: '+endIndex)
                    console.log(' ')
                }
            }
        
           
            
           
        }
        
        
    }
  
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .my-calculator{
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-auto-rows: minmax(50px, auto);
        /* width: 400px; */
        margin: 0 auto;
        font-size: 40px;
    }
    .my-display{
        grid-column: 1/5;
        background-color: #333;
        color: white;
        padding: 10%;
        position: relative;
        height: 25vh;
        text-align: right;
        display: flex;
        align-items: flex-end;
        flex-direction: column-reverse;

    }

    .my-display span{
        display: block;
    }
    /* .my-display span{
        position: absolute;
        bottom: 20px;
    } */
    .zero{
        grid-column: 1/3;
    }

    .btn{
        background-color: #eee;
        border: 1px solid #333;
        padding: 10%;
        cursor: pointer;
    }

    .operator{
        background-color: orange;
        color: white
    }
</style>
