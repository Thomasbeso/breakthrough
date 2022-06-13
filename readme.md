# Breakthrough Format

There are 2 types of breakthrough files, lock files and game files.

## Game files

Games are loaded from a `game1.txt`

```py
def __SetupGame(self):
    """
    Sets up the initial game state from either a pre-initialized file or a random configuration
    """
    Choice = input("Enter L to load a game from a file, anything else to play a new game:> ").upper()
    if Choice == "L":
        if not self.__LoadGame("game1.txt"):
            self.__GameOver = True
```

The game format is, line by line:

1. The current score
2. The current lock in the same format as in `locks`.txt  
   * Challenges are `;` delimited, and within each challenge each condition is `,` delimited
3. A `;` delimited array of `Y` or `N` characters, corresponding to challenges in the lock from line 3, where `Y` means the challenge is met and `N` means the challenge is not met  
   * Lines 4-7 are the current card collections all in the same format
   * Each card is formatted as `{desc} {number}`, where `desc` is either `Dif` or the type and kit of a tool card
   * E.g `Dif 3`, `P a 31`, `K c 4`, joined by `,` to get `Dif 3,P a 31,K c 4`
   * > Note the lack of spaces between the items - JUST a comma
4. The hand card collection
5. The sequence card collection
6. The discard card collection
7. The deck card collection

Code to load the game from `breakthrough.py`:

```py
def __LoadGame(self, FileName):
    """
    Loads the game state from a pre-initialized file at 'FileName'
    """
    try:
        with open(FileName) as f:          
            LineFromFile = f.readline().rstrip()
            self.__Score = int(LineFromFile)
            LineFromFile = f.readline().rstrip()
            LineFromFile2 = f.readline().rstrip()
            self.__SetupLock(LineFromFile, LineFromFile2)
            LineFromFile = f.readline().rstrip()
            self.__SetupCardCollectionFromGameFile(LineFromFile, self.__Hand)
            LineFromFile = f.readline().rstrip()
            self.__SetupCardCollectionFromGameFile(LineFromFile, self.__Sequence)
            LineFromFile = f.readline().rstrip()
            self.__SetupCardCollectionFromGameFile(LineFromFile, self.__Discard)
            LineFromFile = f.readline().rstrip()
            self.__SetupCardCollectionFromGameFile(LineFromFile, self.__Deck)
            return True
    except:
        print("File not loaded")
        return False

def __SetupLock(self, Line1, Line2):
    """
    Sets up a lock from 2 lines of pre-initialized game data
    """
    SplitLine = Line1.split(";")
    for Item in SplitLine:
        Conditions = Item.split(",")
        self.__CurrentLock.AddChallenge(Conditions)
    SplitLine = Line2.split(";")
    for Count in range(0, len(SplitLine)):
        if SplitLine[Count] == "Y":
            self.__CurrentLock.SetChallengeMet(Count, True)

def __SetupCardCollectionFromGameFile(self, LineFromFile, CardCol):
    """
    Builds the card collections from a pre-initialized game file
    """
    if len(LineFromFile) > 0:
        SplitLine = LineFromFile.split(",")
        for Item in SplitLine:
            if len(Item) == 5:
                CardNumber = int(Item[4])
            else:
                CardNumber = int(Item[4:6])
            if Item[0: 3] == "Dif":
                CurrentCard = DifficultyCard(CardNumber)
                CardCol.AddCard(CurrentCard)
            else:
                CurrentCard = ToolCard(Item[0], Item[2], CardNumber)
                CardCol.AddCard(CurrentCard)
```

Idealised simple code to load the game:

```py
def __LoadGame(self, FileName):
    """
    Loads the game state from a pre-initialized file at 'FileName'
    """
    try:
        with open(FileName) as f:
            self.__Score = int(f.readline().rstrip())
            CurrentLockChallenges = f.readline().rstrip().split(";")
            CurrentLockChallengeStatuses = f.readline().rstrip().split(";")

            self.__CurrentLock = Lock()
            CurrentChallenge = 0
            for Challenge, ChallengeStatus in zip(CurrentLockChallenges, CurrentLockChallengeStatuses):
                Conditions = Challenge.split(",")
                self.__CurrentLock.AddChallenge(Conditions)
                
                if ChallengeStatus == "Y":
                    self.__CurrentLock.SetChallengeMet(CurrentChallenge, True)

                CurrentChallenge += 1

            self.__LoadCards(self.__Hand, f.readline().rstrip())
            self.__LoadCards(self.__Sequence, f.readline().rstrip())
            self.__LoadCards(self.__Discard, f.readline().rstrip())
            self.__LoadCards(self.__Deck, f.readline().rstrip())

            return True
    except:
        print("File not loaded")
        return False

def __LoadCards(self, CardCollection, CardLine):
    for CardText in CardLine.split(","):
        CardNumber = int(CardText[4:].strip())
        if CardText.startswith("Dif"):
            Card = DifficultyCard(CardNumber)
        else:
            Type = CardText[0]
            Kit = CardText[2]
            Card = ToolCard(Type, Kit, CardNumber)

        CardCollection.AddCard(Card)
```

## Locks

Locks are stored in `locks.txt` and this is hardcoded in `__LoadLocks` (line 190)

```py
def __LoadLocks(self):
    """
    Loads locks from a pre-initialized file
    """
    FileName = "locks.txt"
```

The lock file has a very simple format:

* Each line is one lock
* Each lock is a `;` delimited list of challenges
* In turn, each challenge is a `,` delimited list of conditions

Code to load each lock:

```py
with open("locks.txt") as f:
    # Iterating a file is equivalent to iterating lines
    FileLocks = []
    for Line in f:
        NewLock = Lock()
        # Split the line into challenges and split each challenge into conditions
        for Challenge in Line.rstrip().split(";"):
            Conditions = Challenge.split(",")
            NewLock.AddChallenge(Conditions)

        FileLocks.append(NewLock)

    # Loaded locks are now in FileLocks
```

Code to save each lock (assuming the locks to save are in `self.__Locks`):

```py
with open("locks.txt", "w") as f:
    for LockToSave in self.__Locks:
        Conditions = []
        for Challenge in LockToSave.GetChallenges():
            # Note: You cannot use Lock.__ConvertConditionToString, as it uses ', ' not ',' as a delimiter
            ConditionString = ",".join(Challenge.GetCondition())
            Conditions.append(ConditionString)
            
        print(Conditions, sep=";", file=f)
        # or
        f.write(";".join(Conditions) + "\n")
```
