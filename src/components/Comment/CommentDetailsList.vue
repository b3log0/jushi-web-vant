<template>
  <div>
    <transition-group name="van-slide-left">
      <div
        :key="item.id"
        v-for="(item) in childComment">

        <div @click.stop="showCommentFocus(item)">
          <defaultComment
            :clickEvent="false"
            ref="commenetRef"
            :parent="comment"
            :comment="item"/>
        </div>

        <CommentDetailsList
          ref="commentDetailsRef"
          v-if="item.children!=null"
          :comment="item"
          :childComment="item.children"
        />
      </div>

    </transition-group>
  </div>
</template>

<script>
    import _ from 'lodash'
    import defaultComment from '@/components/Comment/DefaultComment'

    export default {
        name: 'CommentDetailsList',
        props: {
            comment: Object,
            childComment: {
                type: Array,
                default: () => []
            }
        },
        data () {
            return {}
        },
        methods: {
            /**
             * 调用评论输入框
             */
            showCommentFocus (parent) {
                const searchParent = this.getParentSearch(this.$parent)
                searchParent.showCommentFocus(parent)
            },
            getParentSearch (parent) {
                if (parent.$refs.commentInputRef != null) {
                    return parent
                }
                return this.getParentSearch(parent.$parent)
            },
            // 增加一条评论数据
            pushComment (comment) {
                // FIXME 如果是回复的孙子级别会显示回复祖先级别
                // FIXME 因为他之会推动到直属级别
                // this.childComment.unshift(comment)
            },
            /**
             * 子级评论查询
             */
            childCommentQuery (func) {
                if (this.childComment.length > 0) {
                    return
                }
                // 判断是否支持EventSource
                if (typeof (EventSource) !== 'undefined') {
                    // 长连接方式
                    this.childCommentQueryEvent(func)
                } else {
                    this.$notify('你的浏览时不支持EventSource将采用传统方式')
                    // 如果不支持采用传统的ajax
                    this.childCommentQueryAxios(func)
                }
            },
            /**
             * 子级评论查询(EventSource方式)
             */
            childCommentQueryEvent (func) {
                this.$eventSource('/api/web/comment/commentDetails/SSE/' + this.comment.id, {}, func, (event, source) => {
                    let data = JSON.parse(event.data)
                    this.childComment.push(data)
                })
            },
            /**
             * 子级评论查询(axios方式)
             */
            childCommentQueryAxios (func) {
                this.$axios.get('/api/web/comment/commentDetails/' + this.comment.id)
                    .then(res => {
                        this.childComment = res
                        if (_.isFunction(func)) {
                            func()
                        }
                    })
            }
        },
        created () {
            this.childCommentQuery()
        },
        components: {
            defaultComment
        }
    }
</script>

<style scoped>

</style>
