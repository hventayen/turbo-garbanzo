# turbo-garbanzo
## test

<img width="540" height="360" alt="etst" src="https://github.com/user-attachments/assets/51f76f02-e23c-43c2-9b87-015e6ad1512d" />

text
![testts](https://github.com/user-attachments/assets/6c1d8d03-2c85-4795-a25f-62f29a0fe225)
let oscs = [];
let envs = [];
let noiseOsc;
let noiseEnv;

let melodyOscs = [];
let melodyEnvs = [];
let melodyCounter = 0;


let tick = -16;
let scoreProg = -1;
let noteCounter = 1;
let playFlag = false;


let usedKey = "C#";
let usedScore = "calm";

let textAnimationProg = 0;//Press Space to startのアニメーション用


let nameToMidi = {
  "C1":24,"C#1":25,"Db1":25,"D1":26,"D#1":27,"Eb1":27,"E1":28,"Fb1":28,"E#1":29,"F1":29,"F#1":30,"Gb1":30,
  "G1":31,"G#1":32,"Ab1":32,"A1":33,"A#1":34,"Bb1":34,"B1":35,"Cb1":35,
  "C2":36,"C#2":37,"Db2":37,"D2":38,"D#2":39,"Eb2":39,"E2":40,"Fb2":40,"E#2":41,"F2":41,"F#2":42,"Gb2":42,
  "G2":43,"G#2":44,"Ab2":44,"A2":45,"A#2":46,"Bb2":46,"B2":47,"Cb2":47,
  "B#2":48,"C3":48,"C#3":49,"Db3":49,"D3":50,"D#3":51,"Eb3":51,"E3":52,"Fb3":52,"E#3":53,"F3":53,"F#3":54,"Gb3":54,
  "G3":55,"G#3":56,"Ab3":56,"A3":57,"A#3":58,"Bb3":58,"B3":59,"Cb3":60,
  "B#3":60,"C4":60,"C#4":61,"Db4":61,"D4":62,"D#4":63,"Eb4":63,"E4":64,"Fb4":64,"E#4":65,"F4":65,"F#4":66,"Gb4":66,
  "G4":67,"G#4":68,"Ab4":68,"A4":69,"A#4":70,"Bb4":70,"B4":71,"Cb4":71,
  "B#4":72,"C5":72,"C#5":73,"Db5":73,"D5":74,"D#5":75,"Eb5":75,"E5":76,"Fb5":76,"E#5":77,"F5":77,"F#5":78,"Gb5":78,
  "G5":79,"G#5":80,"Ab5":80,"A5":81,"A#5":82,"Bb5":82,"B5":83,"Cb5":83,
  "B#5":84,"C6":84,"C#6":85,"Db6":85,"D6":86,"D#6":87,"Eb6":87,"E6":88,"Fb6":88,"E#6":89,"F6":89,"F#6":90,"Gb6":90,
  "G6":91,"G#6":92,"Ab6":92,"A6":93,"A#6":94,"Bb6":94,"B6":95,"Cb6":95,
};

let effects = [];

class Effect {
  constructor(x,noteName) {
    this.x = x;
    this.prog = 0;
    this.aliveFlag = true;
    let noteNameToColor = {
      "C":[230,0,18],
      "C#":[243,152,0],
      "Db":[243,152,0],
      "D":[255,251,0],
      "D#":[143,195,31],
      "Eb":[143,195,31],
      "E":[0,153,68],
      "E#":[0,158,150],
      "Fb":[0,153,68],
      "F":[0,158,150],
      "F#":[0,160,233],
      "Gb":[0,104,183],
      "G":[0,104,183],
      "G#":[29,32,136],
      "Ab":[146,7,131],
      "A":[146,7,131],
      "A#":[228,0,127],
      "Bb":[228,0,127],
      "B":[229,0,79],
      "B#":[230,0,18],
      "Cb":[229,0,79],
    }
    this.color = noteNameToColor[noteName];
  }
  next() {
    this.prog += 0.02;
    if (this.prog > 1) {
      this.aliveFlag = false;
    }
  }
  draw() {
    this.r = (1 - (1-this.prog) ** 2) * (width/6);
    this.alpha = ((1-this.prog) ** 2) * 225;
    fill(...this.color,this.alpha);
    noStroke();
    arc(this.x,height/2,this.r*2,this.r*2,0,PI*2);
  }
}


function setup() {
  createCanvas(windowWidth, windowHeight);
  for (let i = 0; i < 5; i++) {
    oscs.push(new p5.Oscillator("triangle"));
    envs.push(new p5.Envelope());
    envs[i].setRange(0.3,0.01);
    envs[i].setADSR(0.001,0,1,0.3);
  }
  for (let i = 0; i < 5; i++) {
    melodyOscs.push(new p5.Oscillator("triangle"));
    melodyEnvs.push(new p5.Envelope());
    melodyEnvs[i].setRange(1,0.01);
    melodyEnvs[i].setADSR(0.01,0,1,0.2);
  }
  noiseOsc = new p5.Noise("pink");
  noiseEnv = new p5.Envelope();
  noiseEnv.setRange(0.5,0.01);
  noiseEnv.setADSR(0.01,0.2,0,0.1);

  setInterval(accompanimentLoop,22);
}

window.addEventListener("keydown",(e) => {
  let key = e.key;
  e.preventDefault();
  if (playFlag) {
    if (!e.repeat) {
      //["a","s","d","f","g","h","j","k","l"]
      //["q","w","e","r","t","y","u","i","o","p"]
      if (
          ["1","2","3","4","5","6","7","8","9","0"].includes(key)) {
        melodyCounter = (melodyCounter + 1) % 5;
        let noteNum = 
          ["1","2","3","4","5","6","7","8","9","0"].indexOf(key);

        let note;
        switch (usedKey) {
          case "C":
            note = ["C4","D4","E4","G4","A4","C5","D5","E5","G5","A5","C6","D6","E6","G6"];
            break;
          case "C#":
            note = ["C#4","D#4","F4","G#4","A#4","C#5","D#5","F5","G#5","A#5","C#6","D#6","F6","G#6"]
            break;
          case "Cm":
            note = ["C4","Eb4","F4","G4","Bb4","C5","Eb5","F5","G5","Bb5","C6","Eb6","F6","G6"]
            break;
        }
        melodyOscs[melodyCounter].freq(midiToFreq(nameToMidi[note[noteNum]]));
        melodyEnvs[melodyCounter].play(melodyOscs[melodyCounter],0.01,0.1);
        let effectX = (width/15) * noteNum + width/6 + width/30;
        effects.push(new Effect(effectX, note[noteNum].slice(0,-1)));

      }
    }
  }else{
    //menu
    let scoreIdx;
    switch (key) {
      case " ":
        playFlag = true;

        for(let i = 0; i < oscs.length; i++) {
          oscs[i].start();
          oscs[i].amp(0);
        }
        for(let i = 0; i < melodyOscs.length; i++) {
          melodyOscs[i].start();
          melodyOscs[i].amp(0);
        }
        noiseOsc.start();
        noiseOsc.amp(0);
        break;
      case "ArrowLeft":
        scoreIdx = scoreTitleList.indexOf(usedScore);
        scoreIdx--;
        if (scoreIdx < 0) {
          scoreIdx = 0;
        }
        usedScore = scoreTitleList[scoreIdx];
        usedKey = scoreKeyList[scoreIdx];
        break;
      case "ArrowRight":
        scoreIdx = scoreTitleList.indexOf(usedScore);
        scoreIdx++;
        if (scoreIdx >= scoreTitleList.length) {
          scoreIdx = scoreTitleList.length - 1;
        }
        usedScore = scoreTitleList[scoreIdx];
        usedKey = scoreKeyList[scoreIdx];
        break;
    }
  }
});

function draw() {
  background(255);
  if (playFlag) {
    if (textAnimationProg < 1) {
      textAnimationProg+= 0.03;
      noStroke();
      let easyOut = 1-((1-textAnimationProg) ** 2);
      let easyIn = textAnimationProg ** 2;
      fill(0,0,0,255-textAnimationProg*225);
      textSize((width/40)*(easyOut*0.5+1));
      textAlign(CENTER,CENTER);
      text("Press Space to start",width/2,height/2+height/12);
    }
    for (let i = 0; i < effects.length; i++) {
      effects[i].next();
      if (effects[i].aliveFlag) {
        effects[i].draw();
      }else{
        effects.splice(i,1);
        i--;
      }
    }
  }else{
    noStroke();
    fill(0);
    textSize(width/40);
    textAlign(CENTER,CENTER);
    let scoreIdx = scoreTitleList.indexOf(usedScore);
    let scoreTitleDisp = usedScore;
    if (scoreIdx > 0) {
      scoreTitleDisp = "<- "+scoreTitleDisp;
    }else{
      scoreTitleDisp = "   "+scoreTitleDisp;
    }
    if (scoreIdx < scoreTitleList.length-1) {
      scoreTitleDisp += " ->";
    }else{
      scoreTitleDisp += "   ";
    }
    
    text(scoreTitleDisp,width/2,height/2-height/12);
    text("Press Space to start",width/2,height/2+height/12);
  }

}

function accompanimentLoop() {
  if (playFlag) {
    tick++;
    if (tick >= 0) {
      noteCounter--;
      if (noteCounter === 0) {
        let score = scoreList[usedScore];
        scoreProg++;
        if (scoreProg >= score.length) {
          scoreProg = 0;
        }
        for (let i = 0; i < score[scoreProg].length-1; i++) {
          if (score[scoreProg][i] === "drum") {
            noiseEnv.play(noiseOsc,0,0.01);
          }
          oscs[i].freq(midiToFreq(nameToMidi[score[scoreProg][i]]));
          envs[i].play(oscs[i],0,0.1);
        }
        noteCounter = score[scoreProg][score[scoreProg].length-1];
      }
    }
  }
}


let scoreTitleList = ["calm","dark","cheerful"];
let scoreKeyList = ["C#","Cm","C"];
let scoreList = {
  "calm":[
      ["A#2","A#3",16],
      ["A#3","C#4","E#4",8],
      ["A#2",16],
      ["A#2",8],
      ["A#3","C#4","E#4",16],

      ["F#2","F#3",16],
      ["F#3","A#3","C#4",8],
      ["F#2",16],
      ["F#2",8],
      ["F#3","A#3","C#4",16],

      ["G#2","G#3",16],
      ["G#3","C4","D#4",8],
      ["G#2",16],
      ["G#2",8],
      ["G#3","C4","D#4",16],

      ["C#3","C#4",16],
      ["C#4","E#4","G#4",8],
      ["C#3",8],
      ["C3","C4",8],
      ["C3",8],
      ["C4","D#4","G#4",16]
    ],
  "dark":[
      ["C3","G3","C4","Eb4",16],
      ["drum","G3","C4","Eb4",11],
      ["C3",5],
      ["G3","C4","Eb4",16],
      ["drum","G3","C4","Eb4",16],
    
      ["C3","G3","Bb3","Eb4",16],
      ["drum","G3","Bb3","Eb4",11],
      ["C3",5],
      ["G3","Bb3","Eb4",16],
      ["drum","G3","Bb3","Eb4",16],
    
      ["Ab2","Ab3","C4","Eb4",16],
      ["drum","Ab3","C4","Eb4",11],
      ["Ab2",5],
      ["Ab3","C4","Eb4",16],
      ["drum","Ab3","C4","Eb4",16],
    
      ["G2","F3","Bb3","Eb4",16],
      ["drum","F3","Bb3","Eb4",11],
      ["G2",5],
      ["F3","Bb3","D4",16],
      ["drum","F3","Bb3","D4",16]
    ],
  "cheerful":[
    ["C4","E4","G4",8],
    ["C4","E4","G4",8],
    ["drum",8],
    ["C4","E4","G4",16],
    ["C4","E4","G4",8],
    ["drum",8],
    ["C4","E4","G4",8],

    ["A3","E4","G4",8],
    ["A3","E4","G4",8],
    ["drum",8],
    ["A3","E4","G4",16],
    ["A3","E4","G4",8],
    ["drum",8],
    ["A3","E4","G3",8],

    ["F3","C4","G4",8],
    ["F3","C4","G4",8],
    ["drum",8],
    ["F3","C4","G4",16],
    ["F3","C4","G4",8],
    ["drum",8],
    ["F3","C4","G4",8],

    ["G3","C4","D4",16],
    ["drum",8],
    ["G3","B3","D4",24],
    ["drum",16]

  ]
}
