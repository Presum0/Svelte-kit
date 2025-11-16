<!-- src/routes/+page.svelte -->
<script>
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';
  
  let decks = [];
  let currentDeck = null;
  let currentCardIndex = 0;
  let showAnswer = false;
  let mode = 'menu';
  let newDeckName = '';
  let newCardFront = '';
  let newCardBack = '';
  let editingDeckIndex = null;

  onMount(() => {
    loadDecks();
  });

  function loadDecks() {
    if (browser) {
      const saved = localStorage.getItem('flashcardDecks');
      if (saved) {
        decks = JSON.parse(saved);
      }
    }
  }

  function saveDecks() {
    if (browser) {
      localStorage.setItem('flashcardDecks', JSON.stringify(decks));
    }
  }

  function createDeck() {
    if (newDeckName.trim()) {
      decks = [...decks, { name: newDeckName, cards: [] }];
      saveDecks();
      newDeckName = '';
      mode = 'menu';
    }
  }

  function deleteDeck(index) {
    if (confirm('Delete this deck?')) {
      decks = decks.filter((_, i) => i !== index);
      saveDecks();
    }
  }

  function startStudying(deck, index) {
    if (deck.cards.length === 0) {
      alert('This deck has no cards yet!');
      return;
    }
    currentDeck = deck;
    editingDeckIndex = index;
    currentCardIndex = 0;
    showAnswer = false;
    mode = 'study';
  }

  function flipCard() {
    showAnswer = !showAnswer;
  }

  function nextCard() {
    if (currentDeck && currentCardIndex < currentDeck.cards.length - 1) {
      currentCardIndex++;
      showAnswer = false;
    }
  }

  function prevCard() {
    if (currentCardIndex > 0) {
      currentCardIndex--;
      showAnswer = false;
    }
  }

  function startEditing(deck, index) {
    currentDeck = deck;
    editingDeckIndex = index;
    mode = 'edit';
  }

  function addCard() {
    if (newCardFront.trim() && newCardBack.trim()) {
      currentDeck.cards = [...currentDeck.cards, { 
        front: newCardFront, 
        back: newCardBack 
      }];
      decks[editingDeckIndex] = currentDeck;
      saveDecks();
      newCardFront = '';
      newCardBack = '';
    }
  }

  function deleteCard(cardIndex) {
    currentDeck.cards = currentDeck.cards.filter((_, i) => i !== cardIndex);
    decks[editingDeckIndex] = currentDeck;
    saveDecks();
  }

  function shuffleDeck() {
    if (currentDeck) {
      currentDeck.cards = currentDeck.cards.sort(() => Math.random() - 0.5);
      decks[editingDeckIndex] = currentDeck;
      saveDecks();
      currentCardIndex = 0;
      showAnswer = false;
    }
  }

  function handleKeyPress(e) {
    if (e.key === 'Enter') {
      createDeck();
    }
  }
</script>

<main>
  <div class="container">
    <header>
      <h1>Flashcards</h1>
    </header>

    {#if mode === 'menu'}
      <div class="section">
        <button class="btn-primary" on:click={() => mode = 'create'}>
          Create New Deck
        </button>

        <div>
          {#if decks.length === 0}
            <p>No decks yet. Create your first one!</p>
          {/if}

          {#each decks as deck, i}
            <div class="deck-item">
              <div>
                <h3>{deck.name}</h3>
                <p>{deck.cards.length} cards</p>
              </div>
              <div class="deck-actions">
                <button on:click={() => startStudying(deck, i)}>Study</button>
                <button on:click={() => startEditing(deck, i)}>Edit</button>
                <button on:click={() => deleteDeck(i)}>Delete</button>
              </div>
            </div>
          {/each}
        </div>
      </div>

    {:else if mode === 'create'}
      <div class="section">
        <h2>Create New Deck</h2>
        <input 
          type="text" 
          placeholder="Deck name" 
          bind:value={newDeckName}
          on:keypress={handleKeyPress}
        />
        <div class="actions">
          <button class="btn-primary" on:click={createDeck}>Create</button>
          <button on:click={() => mode = 'menu'}>Cancel</button>
        </div>
      </div>

    {:else if mode === 'study' && currentDeck}
      <div class="section">
        <div class="header-row">
          <button on:click={() => mode = 'menu'}>Back</button>
          <h2>{currentDeck.name}</h2>
          <button on:click={shuffleDeck}>Shuffle</button>
        </div>

        <div class="card-counter">
          <p>Card {currentCardIndex + 1} of {currentDeck.cards.length}</p>
        </div>

        <div class="flashcard" role="button" tabindex="0" on:click={flipCard} on:keypress={(e) => e.key === 'Enter' && flipCard()}>
          <div>
            {#if !showAnswer}
              <div>
                <div>Question</div>
                <div>{currentDeck.cards[currentCardIndex].front}</div>
              </div>
            {:else}
              <div>
                <div>Answer</div>
                <div>{currentDeck.cards[currentCardIndex].back}</div>
              </div>
            {/if}
          </div>
          <p>Click to flip</p>
        </div>

        <div class="actions">
          <button 
            on:click={prevCard} 
            disabled={currentCardIndex === 0}
          >
            Previous
          </button>
          <button on:click={flipCard}>
            {showAnswer ? 'Show Question' : 'Show Answer'}
          </button>
          <button 
            on:click={nextCard}
            disabled={currentDeck && currentCardIndex === currentDeck.cards.length - 1}
          >
            Next
          </button>
        </div>
      </div>

    {:else if mode === 'edit' && currentDeck}
      <div class="section">
        <div class="header-row">
          <button on:click={() => mode = 'menu'}>Back</button>
          <h2>Edit: {currentDeck.name}</h2>
        </div>

        <div class="add-card">
          <h3>Add New Card</h3>
          <textarea 
            placeholder="Front (Question)" 
            bind:value={newCardFront}
            rows="3"
          ></textarea>
          <textarea 
            placeholder="Back (Answer)" 
            bind:value={newCardBack}
            rows="3"
          ></textarea>
          <button class="btn-primary" on:click={addCard}>Add Card</button>
        </div>

        <div class="cards-list">
          <h3>Cards ({currentDeck.cards.length})</h3>
          {#if currentDeck.cards.length === 0}
            <p>No cards yet. Add your first one!</p>
          {/if}

          {#each currentDeck.cards as card, i}
            <div class="card-item">
              <div>
                <div><strong>Q:</strong> {card.front}</div>
                <div><strong>A:</strong> {card.back}</div>
              </div>
              <button on:click={() => deleteCard(i)}>Delete</button>
            </div>
          {/each}
        </div>
      </div>
    {/if}
  </div>
</main>