<template>
  <div class="testimonial-card card">
    <div class="card-content scrollbar center">
        <router-link class="username" :to="username">
      <img :src="userImg" class="userImage" alt="">
      </router-link>
      <div class="content">
        <h3>{{subject}}</h3>
        <div class="my-2">
          <router-link class="username" :to="username">{{`${username} (${userRep})`}}</router-link>
        </div>
        <router-link class="grey-text text-darken-3" :to="username + '/' + permlink">
        </router-link>
           <router-link class="btn indigo bottom" :to="username + '/' + permlink">View Full Testimonial</router-link>
      </div>
    </div>
    <gig-action :gigData="testimonial" />
  </div>
</template>

<script>
import sc2 from '@/services/sc2'
import GigAction from '@/components/snippets/gigAction'

export default {
  components: {
    GigAction
  },
  props: {
    testimonial: Object
  },
  data () {
    return {
      processing: false
    }
  },
  computed: {
    subject () {
      if (this.testimonial.title) {
        return this.testimonial.title.split('#STEEMGIGS: ')[1]
      } else return ''
    },
    body () {
      return this.testimonial.body
    },
    username () {
      return '@' + this.testimonial.author
    },
    userImg () {
      return this.testimonial.json_metadata.authorPic
    },
    userRep () {
      return this.testimonial.rep
    },
    permlink () {
      return this.testimonial.permlink
    },
    comments () {
      return this.testimonial.children
    },
    upvotes () {
      if (this.testimonial.active_votes) {
        return this.testimonial.active_votes.length
      }
    },
    payout () {
      if (this.testimonial.pending_payout_value) {
        return '$' + this.testimonial.pending_payout_value.amount
      } else {
        return '$' + (parseFloat(this.testimonial.total_payout_value) + parseFloat(this.testimonial.curator_payout_value))
      }
    },
    paymentInfo () {
      if ((new Date(this.testimonial.cashout_time).getTime()) > (new Date().getTime())) {
        return `Will payout in ${Math.floor((new Date(this.testimonial.cashout_time) - (new Date())) / (1000 * 60 * 60 * 24))} days`
      } else {
        return `Author Payout: ${'$' + this.testimonial.total_payout_value}
        Curator Payout: ${'$' + this.testimonial.curator_payout_value}`
      }
    }
  },
  methods: {
    upvote () {
      this.processing = true
      sc2.setAccessToken(this.$store.state.accessToken)
      sc2.vote(this.$store.state.username, this.testimonial.author, this.testimonial.permlink, 10000, (err, res) => {
        this.processing = false
        console.log(err, res)
        if (!err) {
          this.testimonial.active_votes.push({'new': 'vote'})
          console.log(res)
        } else {
          console.log('there was an error voting this\n', 'err:', err)
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .testimonial-card {
    box-shadow: 0 3px 13px rgba(0, 0, 0, 0.15);
    border-radius: 5px;
    transition: all .2s ease-in;
    &:hover {
      box-shadow: 0 3px 30px rgba(0, 0, 0, 0.25);
    }
    .card-content {
        position:relative;
        height: 350px;
        p img {
          max-width: 100%;
        }
    }
    img.userImage {
      height: 90px;
      width: 90px;
      border-radius: 50%;
      margin: auto;
      cursor: pointer;
      box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.3);
    }
    h3 {
      font-size: 1.4rem;
      color: #cfcfcf;
      margin: 7px auto;

    }
    .username {
      color: rgb(120, 79, 149);
      font-size: .9rem;
    }
    p {
      font-size: 1.1rem;
      color: #494646;
      margin: 20px auto;
    }
    .bottom {
      font-size: 13px;
      position: absolute;
      bottom:20px;
      left: 20px;
      right: 20px;
    }
  }
</style>
