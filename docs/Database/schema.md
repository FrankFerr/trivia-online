```DBML
enum matches_type {
  singleplayer
  multiplayer
}

enum matches_status {
  pending
  started
  ended
}

enum difficulty {
  easy
  medium
  hard
  progressive
}

enum answer_status {
  answered
  timeout
}

Table users {
  id uuid [pk]
  username varchar(30) [not null, unique]
  email varchar(50) [not null, unique]
  password varchar(255) [not null]
  is_active bool [not null, default: true]
  created_at timestamp [not null]
}

Table refresh_tokens {
  id uuid [pk]
  id_user uuid [not null, ref: > users.id]
  token varchar(255) [not null, unique]
  id_jwt uuid [not null]
  creation_date date [not null]
  expiry_date date [not null]
  is_used bool [not null, default: false]
  is_revoked bool [not null, default: false]
}

Table categories {
  id uuid [pk]
  description varchar(30)
  is_active bool [not null, default: true]
}

Table questions {
  id uuid [pk]
  id_category uuid [not null, ref: > categories.id]
  text varchar(150)
  explanation varchar(255)
  difficulty difficulty [not null]
  is_active bool [not null, default: true]
}

Table questions_answers {
  id uuid [pk]
  id_question uuid [not null, ref: > questions.id]
  text varchar(255)
  is_correct bool [not null]
}

Table matches {
  id uuid [pk]
  code varchar(20) [not null, unique]
  description varchar(80)
  created_at timestamp [not null, default: "CURRENT_DATE"]
  type matches_type [not null]
  status matches_status [not null, default: "pending"]
  difficulty difficulty
  password varchar(80)
  number_of_players smallint [not null, default: 2]
  number_of_questions smallint [not null]
  answer_timeout_minutes smallint [not null, default: 720]
  current_match_question_id uuid
  updated_at timestamp
  started_at timestamp
  ended_at timestamp
}

Table matches_users {
  id uuid [pk]
  id_user uuid [not null, ref: > users.id]
  id_match uuid [not null, ref: > matches.id]
  is_host bool [not null]
  score integer [default: 0]
}

Table matches_questions {
  id uuid [pk]
  id_match uuid [not null, ref: > matches.id]
  description_category varchar(30) [not null]
  text varchar(150) [not null]
  explanation varchar(255)
  difficulty difficulty [not null]
  position smallint [not null]
}

Table matches_questions_answers {
  id uuid [pk]
  id_match_question uuid [not null, ref: > matches_questions.id]
  text varchar(255) [not null]
  is_correct bool [not null]
}

Table matches_users_answers {
  id uuid [pk]
  id_match_user uuid [not null, ref: > matches_users.id]
  id_match_question uuid [not null, ref: > matches_questions.id]
  id_match_question_answer uuid [not null, ref: > matches_questions_answers.id]
  answered_at timestamp [default: "CURRENT_DATE"]
  status answer_status
}

Table matches_history {
  id uuid [pk]
  description_match varchar(80)
  created_at timestamp [default: "CURRENT_DATE"]
  type matches_type
  difficulty difficulty
  number_of_players smallint
  number_of_questions smallint
  categories varchar(30)[]
  started_at timestamp
  ended_at timestamp
}

Table matches_history_users {
  id uuid [pk]
  id_matches_history uuid [not null, ref: > matches_history.id]
  id_user uuid [not null, ref: > users.id]
  number_of_correct_answers smallint
  score smallint
  is_winner bool
}
```