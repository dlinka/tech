少用!
 
    //错误示范
    if (matchFlag < MATCH_FLAG_LENGTH || !flag) { ... }
    
    //正确示范
    if(matchFlag >= MATCH_FLAG_LENGTH && flag){ ... }
