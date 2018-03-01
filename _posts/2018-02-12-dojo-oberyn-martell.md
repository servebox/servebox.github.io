---
layout: post
title:  "CodingDojo : chaînes de Markov (édition oberyn_martell.rb)"
date:   2018-02-12 07:00:15
tags:
  - meetup
  - coding-dojo
author: fab
thumbnail:
---

***Le retour des Dojo saison 2***

Ce [jeudi 08 février](https://www.meetup.com/fr-FR/Les-evenements-Servebox/events/246711110/) nous avons lancé la nouvelle saison des coding dojos ouverts à tous au sein de [ServeBox](https://servebox.com/).

L'objectif de ce coding dojo était de coder un générateur de mots en français vérifiant la répartition des lettres des mots d'un dictionnaire.

Voilà pour le sujet. En proposant ce sujet, je voulais aborder les sujets suivants :

* Faire un dojo avec la contrainte Navigator-Driver. Le driver sera joué par moi exclusivement et les participants seront les Navigators ;
* Pouvoir se détacher de la théorie mathématique afin de coder simplement un problème ;
* Faire des tests d'un programme utilisant des fonctions aléatoires.

Nous avons codé durant 1h30...

***Code et théorie mathématique***

Le code que nous avons obtenu peut se diviser en 2 parties

* La génération de la première lettre d'un mot. Nous sommes partis un peu rapidement sur l'utilisation du dictionnaire français pour calculer la répartition des premières lettres des mots français. Au final, nous avons une belle structure de données, mais qui est pas très utilisable.
* Pour générer les lettres d'un mot, nous sommes partis sur l'utilisation. Nous avons obtenu un code plus utilisable en ruby et ce code est très éloigné d'une matrice de transformation de Markov.

Certains pourraient reprocher que le code est un _peu_ bourrin, et le fait de stocker 26 tableaux contenant 100 000 éléments n'est pas très performant. Mais l'objectif dans un premier temps n'est pas d'écrire un code performant, mais un code qui fonctionne et qui est lisible.

***Navigator-Driver***

C'est la première fois que je jouais le rôle de driver durant tout un dojo. Et j'avoue que ce n'est pas facile de ne pas pouvoir exprimer son opinion et de suivre les instructions des navigators. D'ailleurs je n'ai pas réussi, il m'est arrivé plusieurs fois de parler sur les solutions proposées. Étant aussi facilitateur, avoir en plus le rôle de driver je ne mettais pas toutes les chances de mon côté pour réussir.

***Test fonction aléatoire***

Je suis moyennement satisfait de notre code de test. En effet, les fonctions pour gérer le choix aléatoire des lettres est beaucoup trop lié à notre structure de données. Ce code n'est pas super maintenable à mon avis.

***Et maintenant ?***

Comme d'habitude, plein de sujets nous ont interpellé lors de cette session. Parmi ces sujets, voici ceux qui ont été laissés en suspens. Peut-être que certaines personnes (moi le premier) présentes ce soir-là auront envie de les aborder seules ou dans une nouvelle session. Je pourrais lister :

* Comment refactorer notre code pour utiliser qu'une seule fonction aléatoire et pas 2 ;
* Et si on utilisait un autre langage (connu pour faire du calcul matricielle R par exemple), le code serait-il différent ?
* Modifier le code pour prendre un livre au format txt sur [Project Gutenberg](https://www.gutenberg.org/browse/languages/fr) et gérer des phrases et non plus des mots.

Il y a sûrement d'autres idées. Je vous laisse maintenant avec notre code

**LE CODE**
```ruby
class Markov
  def initialize(string, random_function: Proc.new { rand },
    sample_function: Proc.new {|array| array.sample})
    @random_function = random_function
    @sample_function = sample_function
    @first_letter_distribution = Hash.new(0)
    @next_letter_distribution = {}
    words = string.split("\n")
    step = 1.0 / words.count
    words.each do |word|
      @first_letter_distribution[word[0]] += step
      word.split('').each_with_index do |char, index|
        end_of_word = word.length == index + 1
        @next_letter_distribution[char] ||= []
        @next_letter_distribution[char]  << (end_of_word ? "\n" : word[index+1])
      end
    end
  end

  def first_letter
    @first_letter_distribution
  end

  def get_first_letter
    percent = @random_function.call()
    sum = 0
    @first_letter_distribution.each_pair do |key, value|
      sum += value
      return key if sum > percent
    end
  end

  def next_letters(letter)
    @next_letter_distribution[letter]
  end

  def get_next_letter(letter)
    @sample_function.call(@next_letter_distribution[letter])
  end

  def get_word
    puts @next_letter_distribution
    word = get_first_letter
    loop do
      next_letter = get_next_letter(word[-1])
      word += next_letter
      break if next_letter == "\n"
    end
    word.split("\n").join
  end
end

describe Markov do
  it 'returns proportion first letter' do
    markov = Markov.new("a\nb")
    expect(markov.first_letter).to eq({'a'=> 0.5, 'b'=> 0.5})
  end

  it 'returns proportion first letter for real' do
    markov = Markov.new("c\nb")
    expect(markov.first_letter).to eq({'c'=> 0.5, 'b'=> 0.5})
  end

  it 'returns proportion first letter for real' do
    markov = Markov.new("c\nb\nc\na")
    expect(markov.first_letter).to eq({'a'=> 0.25, 'b'=> 0.25, 'c' => 0.5})
  end

  it 'returns proportion first letter for real' do
    markov = Markov.new("a\nb\nc\nc", random_function: Proc.new { 0.2 })
    expect(markov.get_first_letter).to eq 'a'
    markov = Markov.new("a\nb\nc\nc", random_function: Proc.new { 0.8 })
    expect(markov.get_first_letter).to eq 'c'
  end

  it 'returns proportion first letter for real' do
    markov = Markov.new("abbibb\ncbb\n")
    expect(markov.next_letters("a")).to eq ["b"]
    expect(markov.next_letters("b")).to eq ["b", "i", "b", "\n", "b", "\n"]
  end

  it 'returns proportion first letter for real' do
    markov = Markov.new("abbibb\ncbb\n",
                        sample_function: Proc.new { |array| array[1] })
    expect(markov.get_next_letter("b")).to eq "i"
  end

  it 'returns a word' do
    markov = Markov.new("abcabc\ncc\n",
                        random_function: Proc.new { 0.1 },
                        sample_function: Proc.new { |array| array[1] })
    expect(markov.get_word).to eq "abc"
  end
end
```

Si vous voulez venir coder avec nous cela se passe à Aix-en-Provence chez [ServeBox](https://servebox.com/) tous les deuxièmes jeudis du mois.
Le prochain : [CodingDojo : Édition Ygritte.*](https://www.meetup.com/fr-FR/Les-evenements-Servebox/events/247081477/)
