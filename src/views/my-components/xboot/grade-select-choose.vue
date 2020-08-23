<template>
  <div>
    <Select v-model="grade" @on-change="$emit('on-change', grade)" placeholder="请选择年级"> 
      <Option v-for="item in gradeList" :value="item.value" :key="item.value">{{ item.label }}</Option>
    </Select>
  </div>
</template>
<script>
export default {
  name: "gradeSelectChoose",
  props: {
      value:{
          type:String,
      }
  },
  data() {
    return {
      grade: "",
      gradeList: [
        {
          value: "2000级",
          label: "2000级",
        },
        {
          value: "2001级",
          label: "2001级",
        },
        {
          value: "2002级",
          label: "2002级",
        },
        {
          value: "2003级",
          label: "2003级",
        },
        {
          value: "2004级",
          label: "2004级",
        },
        {
          value: "2005级",
          label: "2005级",
        },
        {
          value: "2006级",
          label: "2006级",
        },
        {
          value: "2007级",
          label: "2007级",
        },
        {
          value: "2008级",
          label: "2008级",
        },
        {
          value: "2009级",
          label: "2009级",
        },
        {
          value: "2010级",
          label: "2010级",
        },
        {
          value: "2011级",
          label: "2011级",
        },
        {
          value: "2012级",
          label: "2012级",
        },
        {
          value: "2013级",
          label: "2013级",
        },
        {
          value: "2014级",
          label: "2014级",
        },
        {
          value: "2015级",
          label: "2015级",
        },
        {
          value: "2016级",
          label: "2016级",
        },
        {
          value: "2017级",
          label: "2017级",
        },
        {
          value: "2018级",
          label: "2018级",
        },
        {
          value: "2019级",
          label: "2019级",
        },
        {
          value: "2020级",
          label: "2020级",
        },
      ],
    };
  },
  methods:{
      setData(grade){
          this.grade = grade;
          this.$emit("on-change", this.grade);
      }
  }
};
</script>
