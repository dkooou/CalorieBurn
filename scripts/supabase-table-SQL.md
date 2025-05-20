-- 创建用户表：users
create table if not exists users (
  id uuid primary key references auth.users (id) on delete cascade,
  phone text,
  nickname text,
  birthday date,
  gender text,
  height numeric,
  current_weight numeric,
  target_weight numeric,
  fasting_plan jsonb,
  carb_cycle_plan jsonb,
  language text,
  created_at timestamp with time zone default now()
);

-- 创建断食记录表：fasting_records
create table if not exists fasting_records (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  start_time timestamptz not null,
  end_time timestamptz,
  duration numeric,
  is_completed boolean default false,
  created_at timestamptz default now()
);

-- 创建饮食记录表：meal_records
create table if not exists meal_records (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  date date not null,
  calories numeric,
  carbs numeric,
  fats numeric,
  protein numeric,
  is_low_carb_day boolean,
  created_at timestamptz default now()
);

-- 创建运动记录表：activity_records
create table if not exists activity_records (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  date date not null,
  type text,
  duration numeric,
  calories numeric,
  created_at timestamptz default now()
);

-- 创建体重记录表：weight_records
create table if not exists weight_records (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  date date not null,
  weight numeric,
  created_at timestamptz default now()
);

-- 创建计划设置表：plan_configs
create table if not exists plan_configs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  plan_type text, -- fasting / carb_cycle / activity
  config jsonb,
  created_at timestamptz default now()
);

-- 创建提醒设置或记录表：notifications
create table if not exists notifications (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id) on delete cascade,
  type text, -- reminder / push / local
  message text,
  scheduled_at timestamptz,
  is_sent boolean default false,
  created_at timestamptz default now()
);

-- 给 users 表添加 updated_at 字段
alter table users
add column if not exists updated_at timestamptz default now();

-- 给 fasting_records 表添加 notes 字段
alter table fasting_records
add column if not exists notes text;

-- 给 meal_records 表添加 meal_type 字段
alter table meal_records
add column if not exists meal_type text;


-- 为每个表启用 RLS
alter table users enable row level security;
alter table fasting_records enable row level security;
alter table meal_records enable row level security;
alter table activity_records enable row level security;
alter table weight_records enable row level security;
alter table plan_configs enable row level security;
alter table notifications enable row level security;

-- 创建策略
create policy "Users can only access their own data"
on users for all
using (auth.uid() = id);

create policy "Users can only access their own records"
on fasting_records for all
using (auth.uid() = user_id);

create policy "Users can only access their own records"
on meal_records for all
using (auth.uid() = user_id);

create policy "Users can only access their own records"
on activity_records for all
using (auth.uid() = user_id);

create policy "Users can only access their own records"
on weight_records for all
using (auth.uid() = user_id);

create policy "Users can only access their own configs"
on plan_configs for all
using (auth.uid() = user_id);

create policy "Users can only access their own notifications"
on notifications for all
using (auth.uid() = user_id);


-- 为常用查询创建索引
create index if not exists idx_fasting_records_user_date on fasting_records(user_id, start_time);
create index if not exists idx_meal_records_user_date on meal_records(user_id, date);
create index if not exists idx_activity_records_user_date on activity_records(user_id, date);
create index if not exists idx_weight_records_user_date on weight_records(user_id, date);
create index if not exists idx_plan_configs_user_type on plan_configs(user_id, plan_type);
create index if not exists idx_notifications_user_sent on notifications(user_id, is_sent);
