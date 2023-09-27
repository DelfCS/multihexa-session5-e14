
<script>

  // DELFIK LATULIPPE 
  // E14 - Examen Final
  // May 17th 2022

  import '../src/app.css'

  // Différents états du jeux
  const actions = {
    etat_initial: 0,
    mise: 1,
    piege_deux: 2,
    gains: 3,
  }

  // Différents gains possible 
  // gains, nom
  const gains = {
    aucun: [0, 'aucun'],
    pair_above_ten: [1, 'pair above 10'],
    two_pairs: [2, 'two pairs'],
    triple: [3, 'triple'],
    straight: [10, 'straight'],
    flush: [15, 'flush'],
    full_house:[25, 'full housee'],
    four_of_a_kind: [50, 'four of a kind'],
    straight_flush: [100, 'straight flush'],
    royal_flush: [500, 'royal flush - Jackpot !'] 
  }

  // Global Arrays 
  let card_links = new Array();    // Contient les liens vers les imgs
  let player_cards = new Array();  // Main du joueur
  let drawed_cards = new Array();  // Tout les cartes pijées ce tour-ci
  let keep_cards = new Array();    // Représentes le choix que le joueur fera concernant ses cartes ( Garder OU Jeter )

  let action = actions.etat_initial; // État = Initial
  let gain = gains.aucun;
  let play_btn_text = "Bet"; // Play button text (changes with turns)
  let player_chips = 250;
  let bet_amount = 10;

  // GAINS s'ajuste AUTOMATIQUEMENT quand la valeur de gain[0] change ! 
  $: gain_amount = bet_amount * Number(gain[0]); // P

  // CLASSE - Card
  function Card(index){

    let valid_card = false;

    while(!valid_card){
      this.value = ((Math.floor(Math.random() * 13)) + 1); // 1 to 13
      this.color = ((Math.floor(Math.random() * 4)) + 1); // 1 to 4

      // Si la carte est disponible, on recoit true 
      valid_card = validate_card(this);
    }
    // Ajout à la main du joueur & au visuel du jeu
    player_cards[index] = this;
    drawed_cards[index] = this;
    card_links[index] = "../src/assets/cartes/" + this.value + "-" + this.color + ".png";
  }

  function btn_jouer_handler(){
    switch(action){
      case 0: // État initial -> Commencement
        action = actions.mise
        break; 
      case 1: // Commencement -> Pige 2
        commencer_partie();
        play_btn_text = "Keep"
        action = actions.piege_deux;
        break;
      case 2: // Pige2 -> Gains
        continuer_partie();
        verifier_gains();
        play_btn_text = "Gains"
        action = actions.gains;
        break;
      case 3: // Gains -> Commencement
        attributer_gains();
        terminer_partie();
        action = actions.mise;
        play_btn_text = "Bet"
        break;
    }
  }

  function commencer_partie(){

    if(bet_amount > player_chips){
      alert("Not enough chips !")
      action = actions.mise;
    }else{
      // Paiement
      player_chips = player_chips - bet_amount;
      // Pige #1 - 5 cards
      for(let i = 0; i < 5; i++){
        const new_card = new Card(i);
        card_links[i] = "../src/assets/cartes/" + new_card.value + "-" + new_card.color + ".png";
      }
    }
  }

  function continuer_partie(){
    // Pige #2 - X cards
    for(let i = 0; i < 5; i++){
      if(keep_cards[i] != true){
        const new_card = new Card(i);
      }
    }
  }

  function validate_card(new_card){
    // Valid until an indentical card is found
    let valid_card = true;

     //Looping through all 10 possible drawed cards
     for(let index = 0; index < drawed_cards.length; index++){
       if(drawed_cards[index].value == new_card.value && drawed_cards[index].color === new_card.color){
         valid_card = false;
         console.log('Invalide');
       }else{
         console.log('Valide');
       }
     }
    return valid_card;
  }

  function verifier_gains(){
    let sorted_cards = sort_cards(player_cards);
    //alert('Sorted cards are' + sorted_player_cards[0].value + "-" + sorted_player_cards[1].value + "-" + sorted_player_cards[2].value + "-" + sorted_player_cards[3].value + "-" + sorted_player_cards[4].value );

    // PAIR > TEN
    for(let i = 0; i < 4; i++){
      if(sorted_cards[i].value > 10 && sorted_cards[i].value == sorted_cards[i + 1].value){
        // If value is supperior to 10 and the same on both cards
        gain = gains.pair_above_ten
      }
    }
    // ACE PAIR
    if(sorted_cards[0].value == 1 && sorted_cards[1].value == 1){
      gain = gains.pair_above_ten
    }

    // TWO PAIRS
    for(let i = 0; i < 2; i++){
      // Deux pairs collées
      if(sorted_cards[i].value == sorted_cards[i+1].value && sorted_cards[i+2].value == sorted_cards[i+3].value){
        gain = gains.two_pairs
      } 
    }
    // Deux pairs séparées
    if(sorted_cards[0].value == sorted_cards[1].value && sorted_cards[3].value == sorted_cards[4].value){
      gain = gains.two_pairs
    }

    // TRIPLE
    for(let i = 0; i < 3; i++){
      if(sorted_cards[i].value == sorted_cards[i+1].value == sorted_cards[i+2].value){
        gain = gains.triple
      }
    }

    // FLUSH - STRAIGHT FLUSH - ROYAL FLUSH
    let flush_win = true;
    // FLUSH
    for(let i = 0; i<4; i++){
      if (sorted_cards[i].color != sorted_cards[i+1].color){
        flush_win = false;
      }
    }
    if(flush_win){
      let straight_flush_win = true;
      // STRAIGHT FLUSH
      for(let i = 0; i < 4; i++){
        if(sorted_cards[i]+1 != sorted_cards[i+1]){
          let straight_flush_win = false;
        }
      }
      if(straight_flush_win){
        gain = gains.straight_flush;
      }else{
        let royal_flush_win = true
        // ROYAL FLUSH
        if(sorted_cards[0].value == 1 && sorted_cards[1].value == 10){
          for(let i = 1; i < 4; i++){
            if(sorted_cards[i].value +1 != sorted_cards[i+1].value){
              royal_flush_win = false;
            }
          }
        }
        if(royal_flush_win){
          gain = gains.royal_flush;
        }else{
          gain = gains.flush;
        }
      }
    }

    // FULL HOUSE
    if(sorted_cards[0].value == sorted_cards[1].value &&
        sorted_cards[1].value == sorted_cards[2].value &&
        sorted_cards[3]. value == sorted_cards[4].value){
      gain = gains.full_house;
    }
    if(sorted_cards[0].value == sorted_cards[1].value &&
        sorted_cards[2].value == sorted_cards[3].value &&
        sorted_cards[3]. value == sorted_cards[4].value){
      gain = gains.full_house;
    }

    // FOUR OF A KIND
    for(let i = 0; i < 2; i++){
      if(sorted_cards[i].value == sorted_cards[i+1].value &&
           sorted_cards[i+1].value == sorted_cards[i+2].value &&
           sorted_cards[i+2].value == sorted_cards[i+3].value){
        gain = gains.four_of_a_kind;
      }
    }
  }

  function attributer_gains(){
    if(gain[1] != 'aucun'){
      alert('Congrats ! Your hand ' + gain[1] + ' won you ' + gain_amount + ' chips !')
      player_chips += gain_amount; // Qui est tjrs au bon motent ! ( quand gain change, il change )
    }
    else{
      alert('Better luck next time !')
    }
  }

  function terminer_partie(){
      // Réinitialisation des paramètres
      card_links = new Array(5);
      player_cards = new Array(5);
      keep_cards = new Array();
      drawed_cards = new Array();
      gain = gains.aucun;
      bet_amount = 1;
  }
  
  // Homemade SORT 
  function sort_cards(player_cards){
    let smallValue, bigValue
    for(let j = 4; j > 0; j--){
      for(let i = 0; i < j; i++){
        if(player_cards[i].value > player_cards[i+1].value){
          smallValue = player_cards[i+1].value
          bigValue = player_cards[i].value
          player_cards[i].value = smallValue
          player_cards[i+1].value = bigValue
        }
      }
    }
    return player_cards
  }

</script>

<!-- Table -->
<div class="bg_table font-serif grid content-center justify-items-center tracking-widest">
    {#if action == 0}
      <div class="grid justify-items-center">
        <button type="button" on:click={btn_jouer_handler} class="text-gray-200 bg-stone-800 hover:bg-gray-950 focus:outline-none focus:ring-gray-500 rounded-full text-2xl">Start</button>
      </div>
    {:else}
    <div class="grid justify-items-center">
      <div class="flex items-center mb-4">
        <p class="text-5xl mr-4 text-gray-950 text-bold">Credits: {player_chips}</p>
        <img class="chip_img" src="../src/assets/poker-chips.png" alt="">
      </div>
      <div class="cartes grid grid-cols-7 rounded-xl mt-5">
        
        <div/> <!-- For card placement-->
        {#if action == 2 || action == 3}
          {#each card_links as card_link, index}
            <div class="grid justify-items-center">
              <div class="carte" style="background-image: url('{card_link}')"></div>
                <!-- CHECK BOX (bound to keep_cards array) -->
                <input
                  class=" mt-[0.5rem] h-[1.125rem] w-[1.125rem] cursor-none appearance-none rounded-[0.25rem] border-[0.125rem] border-solid border-neutral-300 outline-none before:pointer-events-none before:absolute before:h-[0.875rem] before:w-[0.875rem] before:scale-0 before:rounded-full before:bg-transparent before:opacity-0 before:shadow-[0px_0px_0px_13px_transparent] before:content-[''] checked:border-primary checked:bg-primary checked:before:opacity-[0.16] checked:after:absolute checked:after:-mt-px checked:after:ml-[0.25rem] checked:after:block checked:after:h-[0.8125rem] checked:after:w-[0.375rem] checked:after:rotate-45 checked:after:border-[0.125rem] checked:after:border-l-0 checked:after:border-t-0 checked:after:border-solid checked:after:border-white checked:after:bg-transparent checked:after:content-[''] indeterminate:border-primary indeterminate:bg-primary indeterminate:after:absolute indeterminate:after:ml-[0.2rem] indeterminate:after:mt-[6px] indeterminate:after:w-[0.5rem] indeterminate:after:border-[0.05rem] indeterminate:after:border-solid indeterminate:after:border-white hover:cursor-pointer hover:before:opacity-[0.04] hover:before:shadow-[0px_0px_0px_13px_rgba(0,0,0,0.6)] focus:shadow-none focus:transition-[border-color_0.2s] focus:before:scale-100 focus:before:opacity-[0.12] focus:before:shadow-[0px_0px_0px_13px_rgba(0,0,0,0.6)] focus:before:transition-[box-shadow_0.2s,transform_0.2s] focus:after:absolute focus:after:z-[1] focus:after:block focus:after:h-[0.875rem] focus:after:w-[0.875rem] focus:after:rounded-[0.125rem] focus:after:content-[''] checked:focus:before:scale-100 checked:focus:before:shadow-[0px_0px_0px_13px_#3b71ca] checked:focus:before:transition-[box-shadow_0.2s,transform_0.2s] checked:focus:after:-mt-px checked:focus:after:ml-[0.25rem] checked:focus:after:h-[0.8125rem] checked:focus:after:w-[0.375rem] checked:focus:after:rotate-45 checked:focus:after:rounded-none checked:focus:after:border-[0.125rem] checked:focus:after:border-l-0 checked:focus:after:border-t-0 checked:focus:after:border-solid checked:focus:after:border-white checked:focus:after:bg-transparent indeterminate:focus:after:w-[0.5rem] indeterminate:focus:after:rounded-none indeterminate:focus:after:border-[0.125rem] indeterminate:focus:after:border-b-0 indeterminate:focus:after:border-l-0 indeterminate:focus:after:border-r-0 dark:border-neutral-600 dark:checked:border-primary dark:checked:bg-primary dark:indeterminate:border-primary dark:indeterminate:bg-primary dark:focus:before:shadow-[0px_0px_0px_13px_rgba(255,255,255,0.4)] dark:checked:focus:before:shadow-[0px_0px_0px_13px_#3b71ca]"
                  type="checkbox"
                  disabled={action != 2}
                  value={keep_cards[index]}
                  bind:checked={keep_cards[index]} />
            </div>
          {/each}
        {/if}
      </div>
      <div class="items-center">
        <!-- PLAY BUTTON -->
        <button type="button" on:click={btn_jouer_handler} class="mt-5 text-gray-200 bg-stone-800 hover:bg-gray-950 focus:outline-none focus:ring-gray-500 rounded-full text-2xl">{play_btn_text}</button>
      </div>
      <div>
        <!-- BET AMOUNT -->
        <input 
          type="range" 
          bind:value={bet_amount}
          disabled={action != 1}
          min=1
          max={player_chips} 
          class="w-full h-3 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700 mt-5">
      </div>
      <div>
        <p class="text-3xl text-gray-950 text-bold font-serif">
          Bet : {bet_amount}
        </p>
      </div>
    </div>
    {/if}
  </div>

<style>
  .chip_img{
    width: 60px;
    height: 60px;
    vertical-align: middle;
  }
  :root{
    background: rgb(106,141,182);
    background: radial-gradient(circle, rgba(106,141,182,1) 0%, rgba(32,40,43,1) 100%);
    margin:0; 
    padding: 0;
    font-family: 'Lucida Console', monospace; 
  }
  .bg_table{
    background-image: url("assets/poker-table.png");
    background-repeat: no-repeat;
    background-position: center;
    background-size: 70%;
    background-attachment: fixed;
    min-width: 58vw;
    min-height: 60vh;
    opacity: 0.9;
  }
  .cartes{
    min-height: 212px;
  }
  .carte{
    margin-left: 17px;
    margin-right: 17px;
    margin-top: 40px;
    background-repeat: no-repeat;
    background-size: contain;
    width: 103px;
    height: 145px;
    border-radius: 5px;
    opacity: 0.8;
  }
</style>
