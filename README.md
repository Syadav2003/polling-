# polling-
from django.db import models

# Create your models here.

class Question(models.Model):
	question_text = models.CharField(max_length = 200)
	pub_date = models.DateTimeField('date published')

	def __str__(self):
		return self.question_text


class Choice(models.Model):
	question = models.ForeignKey(Question, on_delete = models.CASCADE)
	choice_text = models.CharField(max_length = 200)
	votes = models.IntegerField(default = 0)

	def __str__(self):
		return self.choice_text
from django.contrib import admin
# Register your models here.
from .models import Question, Choice

# admin.site.register(Question)
# admin.site.register(Choice)

admin.site.site_header = "Pollster Admin"
admin.site.site_title = "Pollster Admin Area"
admin.site.index_title = "Welcome to the Pollster Admin Area"


class ChoiceInLine(admin.TabularInline):
	model = Choice
	extra = 3


class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [(None, {'fields': ['question_text']}), ('Date Information', {
		'fields': ['pub_date'], 'classes': ['collapse']}), ]
	inlines = [ChoiceInLine]


admin.site.register(Question, QuestionAdmin)
from django.template import loader
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Question, Choice

# Get questions and display them


def index(request):
	latest_question_list = Question.objects.order_by('-pub_date')[:5]
	context = {'latest_question_list': latest_question_list}
	return render(request, 'polls / index.html', context)

# Show specific question and choices


def detail(request, question_id):
	try:
		question = Question.objects.get(pk = question_id)
	except Question.DoesNotExist:
		raise Http404("Question does not exist")
	return render(request, 'polls / detail.html', {'question': question})

# Get question and display results


def results(request, question_id):
	question = get_object_or_404(Question, pk = question_id)
	return render(request, 'polls / results.html', {'question': question})

# Vote for a question choice


def vote(request, question_id):
	# print(request.POST['choice'])
	question = get_object_or_404(Question, pk = question_id)
	try:
		selected_choice = question.choice_set.get(pk = request.POST['choice'])
	except (KeyError, Choice.DoesNotExist):
		# Redisplay the question voting form.
		return render(request, 'polls / detail.html', {
			'question': question,
			'error_message': "You didn't select a choice.",
		})
	else:
		selected_choice.votes += 1
		selected_choice.save()
		# Always return an HttpResponseRedirect after successfully dealing
		# with POST data. This prevents data from being posted twice if a
		# user hits the Back button.
		return HttpResponseRedirect(reverse('polls:results', args =(question.id, )))
