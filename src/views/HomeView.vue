<script>
export default {
  data() {
    return {
      presetAmounts: [100, 200, 500, 1000, 5000, 10000, 20000, 50000],
      selectedAmount: 0,
      beneficiaryEmailInput: '',
      beneficiaryEmails: [],
      inputTarget: null,
      missionTitle: '',
      missionImage: '',
      missionDescription: '',
      missions: [],
      imagePreview: null
    };
  },
  watch: {
    beneficiaryEmailInput(newValue) {
      this.beneficiaryEmails = newValue.split(',').map(email => email.trim()).filter(email => email);
    }
  },
  methods: {
    contribute(index) {
      this.missions[index].raised += this.selectedAmount;
      this.missions[index].beneficiaryEmails.push(...this.beneficiaryEmails);
      this.selectedAmount = 0;
      this.beneficiaryEmailInput = '';
    },
    addNewTarget() {
      this.missions.push({
        title: this.missionTitle,
        image: this.missionImage,
        description: this.missionDescription,
        target: this.inputTarget,
        raised: 0,
        beneficiaryEmails: []
      });
      this.missionTitle = '';
      this.missionImage = '';
      this.missionDescription = '';
      this.inputTarget = null;
    },
    selectAmount(amount) {
      this.selectedAmount = amount;
    },
    formatAmount(amount) {
      if (amount >= 1000) {
        return amount / 1000 + 'K';
      }
      return amount;
    },
    handleFileUpload() {
      const file = this.$refs.fileInput.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.imagePreview = e.target.result;
          this.missionImage = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    shareMission(title) {
      if (navigator.share) {
        navigator.share({
          title: `Support our ${title} Crowdfunding Campaign!`,
          text: `Join us in contributing to our ${title} mission.`,
          url: window.location.href
        }).catch((error) => {
          alert('Error sharing: ' + error.message);
        });
      } else {
        alert("Web Share API is not supported on this platform.");
      }
    }
  }
};
</script>

<template>
  <div class="  w-11/12 mx-auto shadow-lg p-3 rounded-md h-[300px]">
    <div class=" relative font-mono text-xs" v-for="(mission, index) in missions" :key="index">
      <div class=" flex justify-between items-center w-11/12 mx-auto">
        <h2 class=" font-mono uppercase text-sm font-bold tracking-widest">{{ mission.title }} -  ${{ mission.target }}</h2>
        <p class=" font-bold font-mono ">Beneficiaries: {{ mission.beneficiaryEmails.length }}</p>
      </div>
      <p class=" my-2 ">{{ mission.description }}</p>
      <div class=" flex justify-center mx-auto items-centerw-11/12">
        
      <div class="preloader ">
        <div class="raised" :style="{width: (mission.raised / mission.target) * 100 + '%'}">
          {{ ((mission.raised / mission.target) * 100).toFixed(2) }}%
        </div>
      </div>
      </div>
      <div class=" titleAndLoader flex justify-between items-center space-x-3">
        <div class=" imageAndDescription w-5/12">
          
          <img class="mission-image" :src="mission.image" alt="Target Mission Image" v-if="mission.image" />
          
        </div>
      
        <div class=" contributionAndCommand w-7/12">
          
          
        <form class=" relative" @submit.prevent="contribute(index)">
          <div class="amount-buttons grid grid-cols-4">
            <button class=" " v-for="amount in presetAmounts" :key="amount" @click.prevent="selectAmount(amount)">
              ${{ formatAmount(amount) }}
            </button>
          </div>
          <p>Selected Amount: ${{ formatAmount(selectedAmount) }}</p>
          <!--
          <div class="email-pills">
            <span v-for="email in beneficiaryEmails" :key="email" class="email-pill">{{ email }}</span>
          </div>
        -->
          <input class=" bg-gray-100 h-8 w-full" v-model="beneficiaryEmailInput" placeholder="Enter comma-separated beneficiary emails" />
         
          <div class=" w-full flex justify-between text-xs absolute -bottom-12">
            <button class=" w-5/12 bg-blue-400 rounded-md tracking-widest text-xs h-8 px-4 uppercase text-white font-mono" :disabled="mission.raised >= mission.target">Contribute</button>
            <button class=" w-5/12 bg-blue-400 rounded-md tracking-widest text-xs h-8 px-4 uppercase text-white font-mono" @click.prevent="shareMission(mission.title)">Share</button>
          </div>
        </form>
      </div>
      </div>
      
      
      <p v-if="mission.raised >= mission.target" class="success absolute -bottom-24 inset-x-0 flex justify-center text-base">Congratulations! Donations complete</p>
    </div>
  </div>



  <div class=" fixed bottom-2 inset-x-0 mx-2">
      <h2 class=" uppercase font-mono text-sm font-bold text-gray-500 w-11/12 mx-auto">Set Fundraising Target</h2>
      <div class=" w-11/12 mx-auto shadow p-3 rounded-md">
        <form class=" space-y-1 flex flex-col" @submit.prevent="addNewTarget">
          <div class=" flex justify-between items-center px-2">
            <input type="file" ref="fileInput" @change="handleFileUpload" accept="image/*" />
            <img class="preview-image" :src="imagePreview" v-if="imagePreview" alt="Preview" />
          </div>
          <input class=" bg-gray-100 h-8" v-model="missionTitle" placeholder="Title of Target Mission" />
          <textarea class=" bg-gray-100" v-model="missionDescription" rows="3" placeholder="Description of Target Mission"></textarea>
          <div class=" flex items-center space-x-2">
            <p>$</p><input class=" bg-gray-100 h-8" v-model.number="inputTarget" type="number" step="any" />
          </div>
          <button class=" bg-blue-400 text-white uppercase font-bold tracking-widest h-12" type="submit">Add Target</button>
        </form>
      </div>
    </div>
</template>




<style>
.preloader {
  background-color: rgb(232, 232, 232);
  height: 20px;
  width: 100%;
  border-radius: 5px;
  overflow: hidden;
}
.raised {
  background-color: rgb(73, 246, 128);
  height: 100%;
  width: 0;
  white-space: nowrap;
  color: white;
  line-height: 20px;
  padding-left: 5px;
}
.amount-buttons button {
  margin: 5px;
  padding: 5px 5px;
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 5px;
  cursor: pointer;
}
.success {
  font-size: 24px;
  color: green;
}
.email-pills {
  margin-top: 10px;
}
.email-pill {
  display: inline-block;
  background-color: #f5f5f5;
  padding: 5px 10px;
  border-radius: 15px;
  margin-right: 5px;
}
.preview-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
}
.mission-image {
  width: 200px;
  height: 150px;
  object-fit: cover;
}
</style>
