## DDL

### 增加字段和索引



```sql
-- 增加字段
ALTER TABLE video_tasks
    ADD `course_id` int unsigned NOT NULL DEFAULT '0' COMMENT '课程id',
    ADD `course_chapter_id` int unsigned NOT NULL DEFAULT '0' COMMENT '课程节id',
    ADD `template_id` int unsigned NOT NULL DEFAULT '0' COMMENT '模板id';

-- 增加索引
ALTER TABLE video_tasks
    ADD INDEX `idx_course_id` (`course_id`) USING BTREE,
    ADD INDEX `idx_course_chapter_id` (`course_chapter_id`) USING BTREE,
    ADD INDEX `idx_user_id` (`user_id`) USING BTREE;
```

