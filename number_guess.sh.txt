#!/bin/bash
PSQL="psql --username=freecodecamp --dbname=number_guess -t --no-align -c"

SECRET_NUMBER=$(( $RANDOM % 1000 + 1 ))

echo "Enter your username:"
read USERNAME

RETURNING_USER=$($PSQL "SELECT username FROM users WHERE username='$USERNAME'")
if [[ -z $RETURNING_USER ]]
then 
  INSERTED_USER=$($PSQL "INSERT INTO users(username) VALUES ('$USERNAME')")
  echo "Welcome, $USERNAME! It looks like this is your first time here."
  INSERT_USERNAME=$($PSQL "INSERT INTO users VALUES('$USERNAME',0,0)")
  GAMES_PLAYED=$($PSQL "SELECT games_played FROM users WHERE username = '$USERNAME'")
  BEST_GAME=$($PSQL "SELECT best_score FROM users WHERE username = '$USERNAME'")
else 
  GAMES_PLAYED=$($PSQL "SELECT games_played FROM users WHERE username = '$USERNAME'")
  BEST_GAME=$($PSQL "SELECT best_score FROM users WHERE username = '$USERNAME'")
  echo "Welcome back, $USERNAME! You have played $GAMES_PLAYED games, and your best game took $BEST_GAME guesses."
fi
echo $SECRET_NUMBER
echo "Guess the secret number between 1 and 1000:"
read GUESS_NUMBER

NEW_GAMES_PLAYED=$(($GAMES_PLAYED+1))
UPATE_GAME_PLAYED=$($PSQL "UPDATE users SET games_played=$NEW_GAMES_PLAYED WHERE username='$USERNAME'")


COUNT=0
((COUNT++))


while ! [[ "$GUESS_NUMBER" =~ ^[0-9]+$ ]]
do
  echo "That is not an integer, guess again:"
  read GUESS_NUMBER
done

while [ $GUESS_NUMBER != $SECRET_NUMBER ]
do
  if ! [[ "$GUESS_NUMBER" =~ ^[0-9]+$ ]]
  then
    echo "That is not an integer, guess again:"
    read GUESS_NUMBER
    ((COUNT++))
  elif [[ $GUESS_NUMBER -gt $SECRET_NUMBER ]]
  then
    echo "It's lower than that, guess again:"
    read GUESS_NUMBER
    ((COUNT++))
  elif [[ $GUESS_NUMBER -lt $SECRET_NUMBER ]]
  then 
    echo "It's higher than that, guess again:"
    read GUESS_NUMBER
    ((COUNT++))
  fi
  
done

if [[ $COUNT < $BEST_GAME ]] || [[ $BEST_GAME = 0 ]]
then
  UPATE_BEST_GAME=$($PSQL "UPDATE users SET best_score=$COUNT WHERE username='$USERNAME'")
fi

echo "You guessed it in $COUNT tries. The secret number was $SECRET_NUMBER. Nice job!"

