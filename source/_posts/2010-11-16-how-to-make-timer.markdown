---
layout: post
title: 如何自己动手做timer
categories:
- NSRunloop
- NSTimer
- SDL
- timer
- 其他技术
- 精确
- 计时器
status: publish
type: post
published: true
comments: true
---
有点标题党，最近在看开源游戏库[SDL](http://www.libsdl.org)，只是因为要用到其中的thread和timer这些东西，所以就顺便看了看源代码，发现timer很精悍，所以分享其中的带代码。

1.首先通过init函数创建一个timer自己的thread（暂且叫timer线程），所以在使用timer之前一点要先调用init函数。
{% codeblock lang:c %}
int SDL_SYS_TimerInit(void)
{
    timer_alive = 1;
    timer = SDL_CreateThread(RunTimer, NULL);
    if (timer == NULL)
        return (-1);
    return (SDL_SetTimerThreaded(1));
}

static int RunTimer(void *unused)
{
    while (timer_alive) {
        if (SDL_timer_running) {
            SDL_ThreadedTimerCheck();
        }
        SDL_Delay(1);
    }
    return (0);
}
{% endcodeblock %}  

<!-- More -->

  
2.添加一个timer, 新建一个维护timer信息的内部struct，并讲这个struct添加到维护有所有在running的timer链表中去。
{% codeblock lang:c %}
static SDL_TimerID SDL_AddTimerInternal(Uint32 interval, SDL_NewTimerCallback callback, void *param)
{
    SDL_TimerID t;
    t = (SDL_TimerID) SDL_malloc(sizeof(struct _SDL_TimerID));
    if (t) {
        t->interval = ROUND_RESOLUTION(interval);
        t->cb = callback;
        t->param = param;
        t->last_alarm = SDL_GetTicks();
        t->next = SDL_timers;
        SDL_timers = t;
        ++SDL_timer_running;
        list_changed = SDL_TRUE;
    }
#ifdef DEBUG_TIMERS
    printf("SDL_AddTimer(%d) = %08x num_timers = %d\n", interval, (Uint32) t,
           SDL_timer_running);
#endif
    return t;
}
{% endcodeblock %}  

  
3.timer线程的唯一工作就是不断地去更新timer的ticks,当发现timer的ticks满足interval的时候就触发timer并讲这个timer从链表中移出。
{% codeblock lang:c %}
void SDL_ThreadedTimerCheck(void)
{
    Uint32 now, ms;
    SDL_TimerID t, prev, next;
    SDL_bool removed;

    SDL_mutexP(SDL_timer_mutex);
    list_changed = SDL_FALSE;
    now = SDL_GetTicks();
    for (prev = NULL, t = SDL_timers; t; t = next) {
        removed = SDL_FALSE;
        ms = t->interval - SDL_TIMESLICE;
        next = t->next;
        if ((int) (now - t->last_alarm) > (int) ms) {
            struct _SDL_TimerID timer;

            if ((now - t->last_alarm) < t->interval) {
                t->last_alarm += t->interval;
            } else {
                t->last_alarm = now;
            }
#ifdef DEBUG_TIMERS
            printf("Executing timer %p (thread = %lu)\n", t, SDL_ThreadID());
#endif
            timer = *t;
            SDL_mutexV(SDL_timer_mutex);
            ms = timer.cb(timer.interval, timer.param);
            SDL_mutexP(SDL_timer_mutex);
            if (list_changed) {
                /* Abort, list of timers modified */
                /* FIXME: what if ms was changed? */
                break;
            }
            if (ms != t->interval) {
                if (ms) {
                    t->interval = ROUND_RESOLUTION(ms);
                } else {
                    /* Remove timer from the list */
#ifdef DEBUG_TIMERS
                    printf("SDL: Removing timer %p\n", t);
#endif
                    if (prev) {
                        prev->next = next;
                    } else {
                        SDL_timers = next;
                    }
                    SDL_free(t);
                    --SDL_timer_running;
                    removed = SDL_TRUE;
                }
            }
        }
        /* Don't update prev if the timer has disappeared */
        if (!removed) {
            prev = t;
        }
    }
    SDL_mutexV(SDL_timer_mutex);
}
{% endcodeblock %}

这个过程也阐述了timer的基本工作原理，所以也证明了timer不能用来作为精确控制，而且在SDL里面timer只能最多精确到10ms。
并且联系这个过程可以联想到Cocoa中的NSTimer，其实NSTimer也是这样被添加到NSRunloop中，然后到时间后就触发。
