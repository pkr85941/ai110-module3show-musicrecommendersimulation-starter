# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name  

Give your model a short, descriptive name.  
Example: **VibeFinder 1.0**  

---

## 2. Intended Use  

Describe what your recommender is designed to do and who it is for. 

Prompts:  

- What kind of recommendations does it generate  
- What assumptions does it make about the user  
- Is this for real users or classroom exploration  

---

## 3. How the Model Works  

Explain your scoring approach in simple language.  

Prompts:  

- What features of each song are used (genre, energy, mood, etc.)  
- What user preferences are considered  
- How does the model turn those into a score  
- What changes did you make from the starter logic  

Avoid code here. Pretend you are explaining the idea to a friend who does not program.

---

## 4. Data  

Describe the dataset the model uses.  

Prompts:  

- How many songs are in the catalog  
- What genres or moods are represented  
- Did you add or remove data  
- Are there parts of musical taste missing in the dataset  

---

## 5. Strengths  

Where does your system seem to work well  

Prompts:  

- User types for which it gives reasonable results  
- Any patterns you think your scoring captures correctly  
- Cases where the recommendations matched your intuition  

---

## 6. Limitations and Bias 

Where the system struggles or behaves unfairly. 

During evaluation (see below), the "Adversarial: Sad but High-Energy Metal" profile
(`genre=metal, mood=sad, energy=0.9`) exposed a real weakness: the top recommendation
was *Broken Mirror*, a metal song whose mood is tagged **aggressive**, not sad. The
system never checks whether a song can satisfy a user's *contradictory* preferences at
once — it just adds up whatever points are available (genre + energy here), so a
genre/energy match can outrank a song that would actually feel right emotionally. In
other words, the scoring rule has no concept of "this user's request doesn't make
sense together," which real recommenders also struggle with but often hide better.
A second bias shows up with `favorite_genre="opera"` (a genre entirely absent from the
catalog): the system quietly falls back to mood+energy matches without ever signaling
"no genre match exists," which could mislead a user into thinking the system
understood their taste when it actually just ignored it. Finally, because
`GENRE_WEIGHT` (2.0) is the single largest score component and only 1–3 songs exist
per genre in this catalog, users with a favorite_genre that overlaps a genre-heavy
song (like the two "Neon Echo" pop tracks) will keep seeing the same artist recur near
the top, which is a small-dataset "filter bubble" more than a true taste signal.

---

## 7. Evaluation  

How you checked whether the recommender behaved as expected. 

I tested five profiles: three realistic ("High-Energy Pop," "Chill Lofi," "Deep
Intense Rock") and two adversarial ones designed to probe edge cases ("Sad but
High-Energy Metal" and "Genre Not In Catalog"). Output for all five, from
`python -m src.main` with the finalized weights (genre +2.0, mood +1.5, energy
closeness up to +1.0, acoustic bonus +0.5), is below.

```
=== High-Energy Pop ===
Profile: {'genre': 'pop', 'mood': 'happy', 'energy': 0.9}

1. Sunrise City by Neon Echo - Score: 4.42
   Because: genre match (+2.0), mood match (+1.5), energy similarity (+0.92)
2. Gym Hero by Max Pulse - Score: 2.97
   Because: genre match (+2.0), energy similarity (+0.97)
3. Rooftop Lights by Indigo Parade - Score: 2.36
   Because: mood match (+1.5), energy similarity (+0.86)
4. Storm Runner by Voltline - Score: 0.99
   Because: energy similarity (+0.99)
5. Isla Nocturna by Ritmo Sol - Score: 0.96
   Because: energy similarity (+0.96)

=== Chill Lofi ===
Profile: {'genre': 'lofi', 'mood': 'chill', 'energy': 0.35, 'likes_acoustic': True}

1. Library Rain by Paper Lanterns - Score: 5.00
   Because: genre match (+2.0), mood match (+1.5), energy similarity (+1.00), acoustic bonus (+0.5)
2. Midnight Coding by LoRoom - Score: 4.93
   Because: genre match (+2.0), mood match (+1.5), energy similarity (+0.93), acoustic bonus (+0.5)
3. Focus Flow by LoRoom - Score: 3.45
   Because: genre match (+2.0), energy similarity (+0.95), acoustic bonus (+0.5)
4. Spacewalk Thoughts by Orbit Bloom - Score: 2.93
   Because: mood match (+1.5), energy similarity (+0.93), acoustic bonus (+0.5)
5. Coffee Shop Stories by Slow Stereo - Score: 1.48
   Because: energy similarity (+0.98), acoustic bonus (+0.5)

=== Deep Intense Rock ===
Profile: {'genre': 'rock', 'mood': 'intense', 'energy': 0.9}

1. Storm Runner by Voltline - Score: 4.49
   Because: genre match (+2.0), mood match (+1.5), energy similarity (+0.99)
2. Gym Hero by Max Pulse - Score: 2.47
   Because: mood match (+1.5), energy similarity (+0.97)
3. Isla Nocturna by Ritmo Sol - Score: 0.96
   Because: energy similarity (+0.96)
4. Riot in My Chest by The Static Kids - Score: 0.95
   Because: energy similarity (+0.95)
5. Broken Mirror by Wraith Choir - Score: 0.93
   Because: energy similarity (+0.93)

=== Adversarial: Sad but High-Energy Metal ===
Profile: {'genre': 'metal', 'mood': 'sad', 'energy': 0.9}

1. Broken Mirror by Wraith Choir - Score: 2.93
   Because: genre match (+2.0), energy similarity (+0.93)
2. Heartbreak Hotel Room by Nadia Cruz - Score: 1.93
   Because: mood match (+1.5), energy similarity (+0.43)
3. Storm Runner by Voltline - Score: 0.99
   Because: energy similarity (+0.99)
4. Gym Hero by Max Pulse - Score: 0.97
   Because: energy similarity (+0.97)
5. Isla Nocturna by Ritmo Sol - Score: 0.96
   Because: energy similarity (+0.96)

=== Adversarial: Genre Not In Catalog ===
Profile: {'genre': 'opera', 'mood': 'happy', 'energy': 0.5}

1. Rooftop Lights by Indigo Parade - Score: 2.24
   Because: mood match (+1.5), energy similarity (+0.74)
2. Sunrise City by Neon Echo - Score: 2.18
   Because: mood match (+1.5), energy similarity (+0.68)
3. Midnight Coding by LoRoom - Score: 0.92
   Because: energy similarity (+0.92)
4. Focus Flow by LoRoom - Score: 0.90
   Because: energy similarity (+0.90)
5. Coffee Shop Stories by Slow Stereo - Score: 0.87
   Because: energy similarity (+0.87)
```

**Profile-to-profile comparisons:**

- **High-Energy Pop vs. Chill Lofi** — completely different top songs and completely
  different feel (upbeat pop vocals vs. quiet lofi loops), which makes sense: the two
  profiles share no genre, no mood, and target opposite ends of the energy scale, so
  every scoring component pushes toward different songs.
- **Deep Intense Rock vs. High-Energy Pop** — both want energy around 0.9, so the
  *runner-up* lists overlap on high-energy tracks like "Gym Hero," but the winner
  differs (rock vs. pop) because genre is weighted the heaviest. This confirms energy
  alone isn't enough to define a "vibe" — genre still anchors the identity of the
  recommendation.
- **Deep Intense Rock vs. Sad-but-High-Energy Metal** — both surfaced songs from the
  "aggressive/intense" cluster at the top, even though one profile asked for `sad`.
  This was the most surprising result: the system doesn't recognize that "metal" +
  "sad" is an unusual combination in real life, and it doesn't try to reconcile the
  conflict — it just maximizes whatever score is available, landing on an aggressive
  song for a self-described "sad" listener.
- **Genre Not In Catalog** — with no genre in the catalog to match "opera," the
  system gracefully falls back to mood/energy-only scoring rather than crashing or
  returning nothing, which is a strength, but it also means the user never finds out
  their stated genre had zero effect on the results.

In plain language: a song like "Gym Hero" keeps showing up for "Happy Pop" listeners
not because the system understands happiness, but because it is a pop song with high
energy — two of the three things this simple scorer actually checks. If two different
users both say "pop" and "high energy," they will get very similar top results even
if their personal sense of "happy" differs, because the system has no way to measure
happiness beyond the `mood` label and the `valence` number it isn't even using yet.

---

## 8. Future Work  

Ideas for how you would improve the model next.  

Prompts:  

- Additional features or preferences  
- Better ways to explain recommendations  
- Improving diversity among the top results  
- Handling more complex user tastes  

---

## 9. Personal Reflection  

A few sentences about your experience.  

Prompts:  

- What you learned about recommender systems  
- Something unexpected or interesting you discovered  
- How this changed the way you think about music recommendation apps  
