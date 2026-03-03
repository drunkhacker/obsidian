rewrite tag filter를 쓰면 tag를 새로 정의할 수 있는데 이때 기존 tag를 남기는 옵션을 쓰면, 기존 태그를 가지는 로그엔트리가 그대로 전달된다. 이옵션을 false로 하면 새로이 만들어진 태그로 태깅된 로그엔트리가 다시 process를 타고, 기존 로그엔트리는 거기서 끝난다.

```
[SERVICE]
    Parsers_File  parsers.conf
    Log_Level debug

[INPUT]
    Name tail
    Path /data/test*.log
    Parser docker
    Tag mytest.*


[FILTER]
    Name rewrite_tag
    Match mytest.*
    #### 주목 ####
    Rule upload_target t1_modem_check t1_modem_check true 
    Emitter_Name re_emitted

[FILTER]
    Name grep
    Match mytest.*
    Regex data_lake_table .*


[OUTPUT]
    Name stdout
    Match mytest.*

[OUTPUT]
    Name file
    Match t1_modem_check
    Path /data/out
```

위에 rewrite_tag filter에 Rule을 보면, 맨 마지막 인자인 Keep 부분을 true로 놓으면, 
Rule에 매칭되는 로그, 예를들면, 
```json
{"data_lake_table":"asdfasdfsadf", "upload_target":"t1_modem_check", "d": 4, "aa":10}
```
의 경우, tag가 `t1_modem_check` 로 바뀌고 다시 process flow를 타고, 기존 로그 역시 밑으로 그대로 내려가서 브랜치가 2개가 되는 효과가 있다.
이 로그의 경우, 밑에 grep filter를 타고 std output까지 도달한다.

또한 새로 태깅된 로그는 밑에 file output 까지 도달한다.

Keep 부분을 false로 놓으면, 원래 로그가 사라지기 때문에 grep filter로 가지 않고, 다만 새로 태깅된 로그가 file output 에 도달하게 된다.

[공식 문서](https://docs.fluentbit.io/manual/pipeline/filters/rewrite-tag) 설명 중 
```
The `rewrite_tag` filter, allows to re-emit a record under a new Tag. Once a record has been re-emitted, the original record can be preserved or discarded.
```
